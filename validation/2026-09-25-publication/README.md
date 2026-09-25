# 上传前复验

从仓库根目录执行：

```bash
./run.sh build
./run.sh test --output results/docs-publication-20260925
```

8 个场景、19 项原生内存测试、在线 C ABI 和 7 项错误拒绝检查通过。
源码包含独立 SMOKE 和可选 AxiMasterStats 监控参数。
机器记录见 [summary.json](summary.json)；完整波形和日志保存在上述本机结果目录。
