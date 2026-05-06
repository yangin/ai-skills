# Framework Hints

各语言 / 框架的最小约定。Codegen 模式才需要读；Spec-only 阶段不读。

## 选择规则

1. 项目已有测试 → 沿用同一框架与命名。
2. 项目无测试 → 按语言主流默认。
3. 用户明确指定 → 服从用户。

## 通用原则

- 一个测试只断言一种行为；多分支用 table-driven / parametrized。
- 反 flaky：注入时钟、注入随机源、mock I/O；不要 `sleep`。
- 用 `t.Parallel()` / 并行模式时，禁止共享全局可变状态。
- 断言信息要能直接定位（输出实际值 + 期望值 + 用例名）。
- 优先沿用项目已安装依赖；不要为了骨架测试新增包、改配置或改 CI，除非用户明确确认。
- 文件系统用例只写临时目录 / sandbox；网络、数据库、队列必须 mock 或 fake。

## Go

- 文件名：`*_test.go`，与被测同包，黑盒测试用 `package foo_test`。
- 函数名：`TestXxx`、子用例 `t.Run("name", ...)`。
- 表驱动：`tests := []struct{ name string; ... }`。
- 并发问题：跑 `go test -race`。
- mock：`gomock` / 接口注入；时钟用 `clock.Clock`。
- 断言：标准库 `t.Fatalf` / `t.Errorf`，需要时 `testify`。

## TypeScript / JavaScript

- 框架：默认 `vitest`，已有 `jest` 沿用。
- 文件名：`*.test.ts` 与被测同目录，或 `__tests__/`。
- 时间：`vi.useFakeTimers()` / `jest.useFakeTimers()`。
- 网络：`msw` 拦截，禁止 hit 真实 URL。
- 断言：`expect().toBe / toEqual / toThrow`。

## Python

- 框架：`pytest`。
- 文件名：`test_*.py`，函数 `test_*`。
- 参数化：`@pytest.mark.parametrize`。
- 时间：`freezegun` / `pytest-freezer`。
- 网络：`responses` / `httpx_mock`。
- fixture：尽量函数级，不要 module 级共享可变状态。

## Java / Kotlin

- 框架：JUnit 5（`@Test`、`@ParameterizedTest`）。
- 时间：注入 `Clock`，测试用 `Clock.fixed`。
- mock：`Mockito` / `MockK`（Kotlin）。
- 断言：`AssertJ`。

## Rust

- `#[cfg(test)] mod tests`，文件内或 `tests/` 目录。
- `#[test]` / `#[tokio::test]`。
- 时间：`tokio::time::pause()` 或自注入。
- mock：trait + 测试替身；外部 HTTP 用 `wiremock`。

## 命名建议

`TC-<编号>_<行为>_<条件>_<期望>`

- `TC-07_createOrder_emptyCart_returnsValidationError`
- `TC-12_expireAt_acrossMidnight_usesInjectedClock`

## 骨架交付要求（Codegen）

- 新增文件，不覆盖既有测试。
- 目标文件已存在时停止，报告建议路径，不追加、不覆盖。
- 顶部注释列出该文件覆盖的 TC 编号与对应 Scout ID。
- 缺 fixture / mock 的位置标 `// TODO: fixture`，并在交付说明里列出。
- 不写需要真实凭证、真实 endpoint 的用例；遇到这种依赖必须 mock 或跳过并说明。
- 项目无测试框架或缺必要依赖时，只输出骨架建议和安装说明，等待用户确认后再改依赖。
