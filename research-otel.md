# 技術研究報告：OpenTelemetry (OTEL)

## 1. 那是什麼 (What is it?)
**OpenTelemetry (簡稱 OTEL)** 是一套開源的觀測標準與工具集。它的核心目標是將 **「可觀測性三支柱」** 統一起來：
*   **Traces (追蹤)**：記錄一個請求從發出到結束，經過了哪些主機與服務（這是解決跨系統除錯的關鍵）。
*   **Metrics (指標)**：數值化的效能數據（如：CPU 使用率、每秒處理請求數）。
*   **Logs (日誌)**：帶有時間戳記的文字記錄。

OTEL 本身不是監控平臺（它不存資料），它是一套 **「搬運標準」**。它確保你的程式可以用統一的方式寫出數據，不論你後端是接 ELK、Grafana 還是 Azure App Insights。

## 2. 怎麼用程式做到 (How to implement in .NET)
在 .NET 環境中，我們主要透過 NuGet 套件來實作。以下是一個典型的配置範例：

### 安裝必要套件
```bash
dotnet add package OpenTelemetry.Extensions.Hosting
dotnet add package OpenTelemetry.Instrumentation.AspNetCore
dotnet add package OpenTelemetry.Instrumentation.Http
dotnet add package OpenTelemetry.Exporter.OpenTelemetryProtocol
```

### 程式碼實作 (Program.cs)
```csharp
using OpenTelemetry.Resources;
using OpenTelemetry.Trace;

var builder = WebApplication.CreateBuilder(args);

// 配置 OpenTelemetry
builder.Services.AddOpenTelemetry()
    .WithTracing(tracerProviderBuilder =>
        tracerProviderBuilder
            .AddSource("MyMISAppName") // 你的應用程式名稱
            .SetResourceBuilder(ResourceBuilder.CreateDefault().AddService("MIS-Core-Service"))
            .AddAspNetCoreInstrumentation() // 自動追蹤傳進來的 Web 請求
            .AddHttpClientInstrumentation() // 自動追蹤傳出去的 API 呼叫 (Trace ID 傳遞關鍵)
            .AddOtlpExporter(opt => 
            {
                opt.Endpoint = new Uri("http://your-otel-collector:4317"); // 送往 Collector
            }));
```

## 3. 有什麼實際意義 (Practical Significance)
在生產環境中，OTEL 解決了 **「黑箱作業」** 的問題：
*   **無痛串接跨系統呼叫**：當你的 B2B 系統呼叫 MIS Core，再呼叫資料庫時，OTEL 會自動在 Header 帶入 `traceparent`。這意味著你可以在 ELK 看到一個請求在不同主機間流轉的全貌。
*   **廠商中立 (Vendor Agnostic)**：以前如果你換了監控廠商，要重寫程式。現在改用 OTEL，你只需要改設定（Exporter），程式碼完全不用動。

## 4. 對你有什麼幫助 (Personal Benefit for MIS Developer)
*   **秒級定位問題**：當 user 抱怨「投保很慢」時，你點開 Tracing 圖表，一眼就能看出是「資料庫查詢太久」還是「外站 API 回傳超時」，不用再一台台主機翻 Log。
*   **大幅減少 Log 程式碼**：透過 Auto-instrumentation，你不用在每個函式開頭寫 `Log.Info("Start...")`，系統會自動幫你記錄進出時間與結果。
*   **技術升級感**：從單純的日誌收集轉向現代化的「分散式追蹤」，這能顯著提升你在團隊中的技術主導地位。

## 5. 專案記錄 (GitHub Documentation)
此研究結果已同步更新至 GitHub 專案 `apm-study-notes` 的 `research-otel.md`。
