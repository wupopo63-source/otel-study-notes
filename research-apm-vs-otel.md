# 技術研究報告：傳統 APM vs OpenTelemetry (OTEL)

## 1. 那是什麼 (What is it?)
*   **APM (Application Performance Monitoring)**：是一個「功能領域」，指監控應用程式效能的行為。
*   **傳統 APM Agent**：通常指廠商私有的工具（如 Elastic APM Agent）。它高度封裝，安裝簡單但具有「廠商鎖定 (Vendor Lock-in)」的特性。
*   **OpenTelemetry (OTEL)**：是一個「標準協定」。它定義了數據的格式與傳輸方式，讓應用程式具備「中立」的觀測能力。

## 2. 怎麼用程式做到 (Code Comparison)

### 傳統廠商 Agent 模式 (以 Elastic 為例)
```csharp
// 需要依賴廠商特定的 SDK
app.UseElasticApm(Configuration);
```

### OpenTelemetry 標準模式 (中立模式)
```csharp
// 依賴通用 API
builder.Services.AddOpenTelemetry()
    .WithTracing(t => t
        .AddAspNetCoreInstrumentation()
        .AddOtlpExporter()); // 使用通用協定傳輸
```

## 3. 有什麼實際意義 (Practical Significance)
*   **靈活性**：OTEL 允許你隨時更換後端存儲（如從 Elasticsearch 換到 ClickHouse），而不必修改應用程式原始碼。
*   **一致性**：跨語言（C#, Java, Python）的監控數據格式統一，易於集中分析。
*   **原生支持**：.NET 核心庫（如 `System.Diagnostics`）已原生支持 OTEL 概念，效能損耗極低。

## 4. 對你有什麼幫助 (Personal Benefit for MIS)
*   **簡化技術債**：不用學習每家廠商不同的 SDK 用法，學會 OTEL 就能應付所有現代監控需求。
*   **提升系統穩定性**：避免傳統 Agent 動態注入可能帶來的死機或效能黑洞風險。
*   **符合企業轉型趨勢**：在邁向容器化與雲原生的過程中，OTEL 是業界公認的唯一標準。

## 5. 專案記錄 (GitHub Documentation)
此筆記已記錄於 `research-apm-vs-otel.md`。
