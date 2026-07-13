# Retry & Rollback Policy

## 超时配置引用

各 Stage 的超时阈值和重试策略定义在 `rules/timeout-retry-config.yaml` 中。执行前需读取该配置文件，按 Stage 类型应用对应的超时和重试规则。

### 超时配置结构
每个 Stage 配置包含以下字段：
- `timeout_seconds`: 最大执行时间（秒）
- `max_retries`: 最大重试次数
- `retry_delay_seconds`: 重试间隔（秒）
- `allow_skip`: 是否允许跳过
- `fallback_strategy`: 超时后的 fallback 策略

### Fallback 策略说明
- `manual`: 停止流程，等待人工介入
- `built-in-chat`: 使用内置聊天能力继续执行
- `default-value`: 使用默认值继续执行
- `skip`: 跳过当前阶段，继续后续流程
- `partial-generation`: 保留已生成代码，跳过失败模块
- `skip-with-warning`: 跳过并记录警告

## Stage 3 失败处理分级

### Level 1: 单模块单元测试失败

**触发**: 某个后端模块生成代码后,该模块单元测试失败。

**处理**:
1. AI 自动诊断(读取测试日志,定位错误)
2. 修复代码
3. 重跑该模块测试
4. 最多重试 **3 次**
5. 3 次仍失败 → 进入 Level 3

**回滚范围**: 仅回退该模块代码到上一个 commit,**不**影响其他模块。

```bash
git checkout HEAD -- <module-path>
```

### Level 2: 冒烟测试失败

**触发**: 所有代码生成完毕,运行冒烟测试时有用例失败。

**处理**:
1. AI 自动诊断(对比预期与实际,定位失败原因)
2. 修复代码(优先修代码,其次修测试用例——但修测试用例需用户同意)
3. 重跑失败用例(不重跑已通过的)
4. 最多重试 **3 次**
5. 3 次仍失败 → 进入 Level 3

**回滚范围**: 回退到 `pre-smoke-test` tag,**不**丢弃已生成代码,只是重新修复后再跑。

```bash
git tag pre-smoke-test   # 测试前打 tag
# 失败后
git reset --hard pre-smoke-test
# 修复后重新生成 + 重新测试
```

### Level 3: 人工介入(对话式)

**触发**: Level 1 或 Level 2 重试 3 次仍失败。

**处理**:
1. AI 输出诊断报告:
   - 失败的具体用例/模块
   - 已尝试的修复方案及失败原因
   - 疑似根因(代码 bug / 测试用例错误 / 设计缺陷)
2. 使用 `AskUserQuestion` 询问用户方向:
   - 选项 A: 用户给出修复方向,AI 继续尝试
   - 选项 B: 用户手动修复后,AI 重跑测试
   - 选项 C: 跳过该失败项,继续后续(标注为 known issue)
   - 选项 D: 回退到 Gate 4,重新审视设计

## 不可自动回滚的情况

以下情况**禁止**自动 git rollback,必须等用户确认:
- 跨 gate 边界回退(如 Stage 3 失败想回退到 Stage 2)
- 回退会丢弃用户手动修改过的文件
- 回退影响超过 3 个模块

## Fallback 计数

`docs/workflow-state.yaml` 中 `stage3.smoke_test.attempts` 记录冒烟测试尝试次数。
每次 Level 2 重试都递增。达到 3 后强制进入 Level 3。

## 代码 vs 测试用例的修复优先级

冒烟测试失败时,默认按以下顺序判断:
1. **优先怀疑代码** (80% 情况是代码 bug)
2. 如果代码明显符合设计,再检查测试用例是否有错
3. 修改测试用例**必须**用户同意(因为测试用例是验收标准,改测试=改标准)

## 所有 Stage 的通用超时与重试规则

### 超时检测流程
1. 在 Stage 开始时记录 `started_at` 时间戳
2. 定期检查执行时间是否超过 `timeout_seconds`
3. 超时触发时,按配置的 `fallback_strategy` 执行:
   - 记录超时事件到 `execution_log`
   - 更新 Stage 状态为 `timeout`
   - 执行 fallback 策略

### 重试执行流程
1. Stage 失败时,检查 `max_retries` 是否大于 0
2. 如果允许重试:
   - 等待 `retry_delay_seconds` 秒（支持指数退避）
   - 重新执行 Stage
   - 递增重试计数
3. 达到最大重试次数后,执行 `fallback_strategy`

### 全局规则
- 总重试次数不超过 `global_rules.max_total_retries`
- 重试间隔按 `retry_backoff_multiplier` 递增
- 执行时间超过 `alert_threshold_seconds` 时发送警告
- 超时可延长 `timeout_extension_factor` 倍（需用户确认）

### Stage 特定规则

**Stage 1 (需求确认):**
- 超时策略: 使用 built-in-chat 继续
- 最大重试: 2 次
- 特殊处理: 保留已收集的需求信息

**Stage 1.3/1.5 (项目类型/技术栈选择):**
- 超时策略: 使用默认值继续
- 最大重试: 1-2 次
- 特殊处理: 默认值记录到 state 文件

**Stage 1.6 (成本评估):**
- 允许跳过: 是
- 超时策略: 跳过继续
- 特殊处理: 标记为 skipped

**Stage 2 (产出物生成):**
- 超时策略: 使用 built-in-chat 继续
- 最大重试: 2 次
- 特殊处理: 保留已生成的部分内容

**Stage 3 (代码生成):**
- 超时策略: partial-generation（保留已生成模块）
- 最大重试: 3 次
- 特殊处理: 按模块粒度重试

**Smoke Test:**
- 允许跳过: 是
- 超时策略: skip-with-warning
- 最大重试: 3 次
