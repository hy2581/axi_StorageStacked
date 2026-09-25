# 独立 AXI 存储验收（2026-09-25）

实际执行：`./run.sh build`，`./run.sh test --output results/split-acceptance-v2`。
验收摘要：[validation/2026-09-25-split/summary.json](validation/2026-09-25-split/summary.json)。
完整 VCD、Flit 和内存日志保留在本机 `results/split-acceptance-v2/`。

- 19/19 原生内存测试通过，在线 C ABI 掩码/队列/大地址窗口检查通过。
- default、slow、backpressure、replay、decode_error、read_only、write_only 共 7 场景通过。
- 重放场景完成 192 笔实际 AXI 请求；逐字节回读、VCD/CSV、Flit、内存子请求、DRAM/DFI 验收全部通过。
- 补充 `examples/edge_cases.json` 已实际运行：覆盖所有 AxSIZE、零掩码写入和 256 拍突发，共 20 笔事务通过。
- 6 类非法输入被拒绝，被篡改的读返回被独立校验拒绝。
- 相同首笔写操作的往返延迟在内存周期从 250,000 fs 增到 1,000,000 fs 后，由 84 ns 增到 144 ns。
  短序列的后续请求可能落入不同刷新窗口，因此整个序列的总耗时不保证单调；未把这一点隐藏为整段加速/减速结论。
- 从仅含本项目的本地 Git 克隆在工作区外全新构建，独立读写通过；该目录没有任一处理器驱动。
  结果：[isolated-build-run.json](validation/2026-09-25-split/isolated-build-run.json)。

重新验收使用新目录：

```bash
./run.sh setup
./run.sh build
./run.sh test --output results/my-acceptance
```

要求总摘要和所有场景 `passed: true`。模型范围和 AXI 子集见 README.md / INTERFACE.md。
PHY/DFI 为行为模型；这是仿真功能与时序反馈验收，不是硬件或工艺签核。

## 上传前复验（2026-09-25）

本次重新执行 `./run.sh build` 和
`./run.sh test --output results/docs-publication-20260925`。
含 SMOKE 的 8 个场景、19 项原生内存测试、在线 C ABI 和 7 项错误拒绝检查均通过。
复验覆盖新增的可选主端统计接口，独立运行器继续使用不传统计参数的调用方式。
摘要见 [上传前回归结果](validation/2026-09-25-publication/summary.json)。
完整波形保存在本机对应的 `results/` 下。

## 输入输出文档与 SMOKE 补充验收（2026-09-25）

新增 [输入说明](docs/INPUTS.md)、[输出说明](docs/OUTPUTS.md) 和统一入口 `./run.sh smoke`。
输出文档第 6 节逐文件说明改动，并区分源码改动与 SMOKE 对仿真内存的写入。

实际执行：

```bash
./run.sh build
./run.sh smoke --output results/smoke-20260925-docs
./run.sh test --output results/acceptance-20260925-smoke
```

- SMOKE 六笔事务通过：AW/W/B/AR/R 握手分别为 2/3/2/4/7。
- 窄拍 `WSTRB=0x50000000`，只更新两个字节，最终回读为 `aa1dcc1f`。
- 实际观测 B/R 反压；在线内存完成 10 个子请求，RD=7、WR=3，命令／DFI 错误为零。
- 最终 64 字节镜像正确，最后一笔响应完成于 676 ns。
- 含新增 SMOKE 的八场景回归、19 项原生测试、C ABI 检查及非法输入／损坏返回拒绝检查全部通过。
- 另行验证旧目录保护、禁用 Python 断言拒绝，以及掩码覆盖被移除后两个摘要均标记失败；
  记录见 [negative_checks.json](validation/2026-09-25-smoke/negative_checks.json)。

关键记录保存在 [validation/2026-09-25-smoke](validation/2026-09-25-smoke/)，
完整波形和原始日志在上面的本机 results 目录。已有目录不能覆盖；
重跑用 `./run.sh smoke`、`./run.sh test` 自动生成新目录，或提供新的 `--output`。
SMOKE 的 `summary.json.passed`、`summary.json.smoke_passed` 和 `smoke_summary.json.passed`
必须同时为 true；具体覆盖边界见输出文档第 5 节。

## 从零添加 SMOKE 教程核验（2026-09-25）

[开发教程](docs/03-从零添加SMOKE测试.md) 现已精简为八步操作清单，完整实现直接查看源码文件。
以下保留教程初版中完整代码与拒绝示例的实际核验记录。

本次实际执行 setup/build、roundtrip 基线、六笔输入的通用 run、直接 Python SMOKE 和 shell SMOKE；
正向验收均通过，SMOKE 为 6 笔事务、18 次握手、10 个内存子请求，末笔响应在 676 ns 完成。
直接从教程提取并执行了读结果、错误 expected、移除部分写覆盖三段代码；两个错误用例均按预期被拒绝。
本次只添加文档与导航／核验记录，未修改已完成的 SMOKE 实现，也未重新执行八场景完整回归。

证据见 [verification.json](validation/2026-09-25-smoke-tutorial/verification.json)，
完整原始记录在 `results/tutorial-smoke-from-zero-*`。负例目录中的 passed=false 是预期结果，
核验摘要单独记录它们确实被拒绝。

## Vortex 用户目录重构的公共接口复验（2026-09-25）

为 gem5 集成添加可选 UCIe 编译参数读取，并在 SCons 中提供公共 AXI 监测器实现。
默认参数不变；Vortex 的新平台已使用同一公共源码完成 gem5 编译。

实际执行 `./run.sh test --output results/vortex-refactor-20260925`：8 个场景、19 项原生内存测试、
在线 C ABI、7 项非法输入及损坏返回拒绝检查通过，慢内存时延反馈通过。
摘要见 [本次复验](validation/2026-09-25-vortex-layout/summary.json)。
