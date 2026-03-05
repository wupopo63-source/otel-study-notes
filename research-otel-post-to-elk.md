# 技術研究報告：OpenTelemetry (OTEL) 擷取 POST 資訊並送往 ELK

## 1. 那是什麼 (What is it?)
在 OpenTelemetry 的標準實作中，預設僅記錄 HTTP Metadata（如方法、URL）。若要記錄 **POST Body (Payload)**，則需要透過 **Enrichment (數據強化)** 機制。這是在 Trace 產生的生命週期中，攔截請求串流並將內容轉存為 Trace Attribute (Tag) 的過程。

## 2. 怎麼用程式做到 (Code Implementation)

### .NET 實作範例 (C#)
在 `Program.cs` 或 `Startup.cs` 中配置：

```csharp
builder.Services.AddOpenTelemetry()
    .WithTracing(tracing => tracing
        .AddAspNetCoreInstrumentation(options =>
        {
            // 使用 Enrich 鉤子擷取 Body
            options.EnrichWithHttpRequest = async (activity, request) =>
            {
                if (request.Method == "POST")
                {
                    // 必須啟用 Buffering 否則 Body 只能讀一次
                    request.EnableBuffering(); 
                    using (var reader = new StreamReader(request.Body, leaveOpen: true))
                    {
                        var body = await reader.ReadToEndAsync();
                        request.Body.Position = 0; // 重置串流位置
                        
                        // 將內容限制長度後存入 Tag (避免過大影響效能)
                        activity.SetTag("http.request.body", body.Length > 1000 ? body.Substring(0, 1000) : body);
                    }
                }
            };
        })
        .AddOtlpExporter()); // 送往 OTEL Collector
```

### 數據流向 (Pipeline)
1. **App**: 擷取 Body 並透過 OTLP 協定送出。
2. **OTEL Collector**: 接收 OTLP 數據，進行過濾處理。
3. **Logstash/Elasticsearch**: 接收數據並存儲至索引。

## 3. 有什麼實際意義 (Practical Significance)
*   **精準除錯**：當 API 回傳 500 錯誤時，能立即查看到底是哪個參數（POST Body）導致的，不需再透過模擬請求重現 Bug。
*   **業務稽核**：可以從效能監控介面直接看到重要的業務參數（如保單 ID、客戶編號），將技術指標與業務資料結合。

## 4. 對你有什麼幫助 (Personal Benefit for MIS)
*   **提升解決問題速度**：對於 MIS 工程師，最痛苦的是 user 說「有問題」但沒給參數。現在所有 Payload 都在 ELK 上，你可以直接查閱「案發現場」。
*   **簡化維運工作**：不再需要為了記錄 Request 而在每個 API Controller 寫重複的日誌代碼，OTEL 一次性配置後全局生效。

## 5. 專案記錄 (GitHub Documentation)
此報告已存檔於 `research-otel-post-to-elk.md`。
