# Why metrics?
[Daniel Berman](https://logz.io/blog/logs-or-metrics/):
> Metrics help measure component functionality and define thresholds for attention-required usage. Metrics give DevOps engineers the ability to assess service value over time and provide a continuous view of the whole environment. There is an infinite number of metrics that can be used to evaluate an application, so it is important to specify the business-critical functionality and build the metrics plan accordingly.
類似KPI (Key Performance Indicator)的概念，測量關鍵的使用情況或效能門檻。

- logs 就是阿貓阿狗都記，適用debug
- metrics 像是 KPI，只記關鍵的指標，適用長期分析，缺點是記錄的同時，可能會耗系統資源(因為要長期計算或統計)；
- tracing 像是柯南，針對特定問題找線索時，需要記錄一定時間內的執行過程
