# Bug Reproduction

## 包的性质

当前 test_model_fix 保存的是被测模型修复后的结果源码，不是初始含 Bug 源码。要复现原始缺陷，必须检出下面固定的 parent SHA；不要在当前修复结果源码上期待重新出现修复前失败。生成系统使用的可信验证补丁和完整验证日志仅在本地留存，不提交到结果分支。

## 问题现象

调用方传入的 X-Request-ID 没有出现在响应和下游日志关联字段中，每次请求都得到新的 ID。请先不要修改代码，定位请求上下文传播丢失的位置并给出证据。

## 含 Bug 版本

- 仓库：zhanglei10281852-gif/t63-qa-17
- 仓库地址：https://github.com/zhanglei10281852-gif/t63-qa-17.git
- parent SHA：404e1f034fc36a3f8d8c9c5956ed7a1c52853342

## 复现步骤

```bash
git clone -- https://github.com/zhanglei10281852-gif/t63-qa-17.git bug-repro
cd bug-repro
git checkout --detach 404e1f034fc36a3f8d8c9c5956ed7a1c52853342
go test ./internal/middleware -run TestRequestIDUsesIncomingValueAndGeneratesMissingValue -count=1
```

## 双架构完整错误信息

### linux/amd64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/middleware -run TestRequestIDUsesIncomingValueAndGeneratesMissingValue -count=1
--- FAIL: TestRequestIDUsesIncomingValueAndGeneratesMissingValue (0.00s)
    middleware_test.go:27: incoming request id was not propagated
FAIL
FAIL	sanitation-operations/internal/middleware	0.003s
FAIL

```

stderr：

```text
warning: internal/middleware/middleware_test.go has type 100755, expected 100644
warning: internal/middleware/middleware_test.go has type 100755, expected 100644

```

### linux/arm64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/middleware -run TestRequestIDUsesIncomingValueAndGeneratesMissingValue -count=1
--- FAIL: TestRequestIDUsesIncomingValueAndGeneratesMissingValue (0.01s)
    middleware_test.go:27: incoming request id was not propagated
FAIL
FAIL	sanitation-operations/internal/middleware	0.172s
FAIL

```

stderr：

```text
warning: internal/middleware/middleware_test.go has type 100755, expected 100644
warning: internal/middleware/middleware_test.go has type 100755, expected 100644

```

## 通过条件

在触发条件下，定向测试 TestRequestIDUsesIncomingValueAndGeneratesMissingValue 应通过，相关包、全量测试、竞态测试和构建检查均通过；回退 gold 唯一修复后定向测试重新失败。
