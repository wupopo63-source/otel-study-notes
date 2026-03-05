# OpenTelemetry (OTEL) 學習與實作筆記

這裡專門存放關於 **OpenTelemetry (OTEL)** 的技術研究與實作指南。

## 目錄
*   [OpenTelemetry (OTEL) 基礎研究](research-otel.md) - 包含定義、.NET 實作範例與實際意義。
*   [傳統 APM vs OpenTelemetry](research-apm-vs-otel.md) - 深入對比兩者的差異與選擇建議。
*   [擷取 POST Body 資訊至 ELK](research-otel-post-to-elk.md) - 如何透過 OTEL Enrichment 機制記錄請求內容。

## 自動化工具
*   [.github/workflows/pack-otel.yml](.github/workflows/pack-otel.yml) - 用於離線環境的 NuGet 套件自動打包腳本。
