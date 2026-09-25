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
