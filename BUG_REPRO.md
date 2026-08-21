# Bug Reproduction

## 包的性质

当前 test_model_fix 保存的是被测模型修复后的结果源码，不是初始含 Bug 源码。要复现原始缺陷，必须检出下面固定的 parent SHA；不要在当前修复结果源码上期待重新出现修复前失败。生成系统使用的可信验证补丁和完整验证日志仅在本地留存，不提交到结果分支。

## 问题现象

两条延迟到达的遥测记录，其 `recorded_at` 相差 45 分钟，漂移事件的首末时间却只覆盖它们进入服务的那几分钟，值域和计数均正常。请修复漂移事件采用的时间来源。已有回归和断言保持不变，禁止修改测试用例；不要跳过完整回归，也不要降低时间范围的检查标准。

## 含 Bug 版本

- 仓库：zhanglei10281852-gif/ai-28
- 仓库地址：https://github.com/zhanglei10281852-gif/ai-28.git
- parent SHA：721ef31a98065c1e3a0e539d130921b29f637920

## 复现步骤

```bash
git clone -- https://github.com/zhanglei10281852-gif/ai-28.git bug-repro
cd bug-repro
git checkout --detach 721ef31a98065c1e3a0e539d130921b29f637920
go test ./internal/service -run ^TestDriftIncidentKeepsTelemetryEventTimes$ -count=1
```

## 双架构完整错误信息

### linux/amd64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/service -run ^TestDriftIncidentKeepsTelemetryEventTimes$ -count=1
--- FAIL: TestDriftIncidentKeepsTelemetryEventTimes (0.64s)
    annotation_intake_behavior_test.go:120: incident event window = 2026-08-18 08:00:00 +0000 UTC..2026-08-18 08:05:00 +0000 UTC, want 2026-08-18 06:00:00 +0000 UTC..2026-08-18 06:45:00 +0000 UTC
FAIL
FAIL	github.com/zhanglei10281852-gif/ai/internal/service	0.642s
FAIL

```

stderr：

```text
(empty)
```

### linux/arm64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/service -run ^TestDriftIncidentKeepsTelemetryEventTimes$ -count=1
--- FAIL: TestDriftIncidentKeepsTelemetryEventTimes (1.27s)
    annotation_intake_behavior_test.go:120: incident event window = 2026-08-18 08:00:00 +0000 UTC..2026-08-18 08:05:00 +0000 UTC, want 2026-08-18 06:00:00 +0000 UTC..2026-08-18 06:45:00 +0000 UTC
FAIL
FAIL	github.com/zhanglei10281852-gif/ai/internal/service	1.473s
FAIL

```

stderr：

```text
(empty)
```

## 通过条件

漂移事件的首次和末次时间必须取自遥测记录各自的 recorded_at，使相差 45 分钟的延迟数据仍呈现完整事件范围；值域、样本计数及其他聚合结果保持原有正确性。定向服务用例与相关漂移回归须通过，不得修改测试或降低时间范围断言。
