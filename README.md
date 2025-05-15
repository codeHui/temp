HKPC-DTD: openai的function calling怎麼包裝自定義的json返回，給我個例子
 
GitHub Copilot: Function Calling 讓您能夠讓模型依照特定結構輸出格式化的 JSON 響應。以下是一個使用 OpenAI Function Calling 包裝自定義 JSON 返回的例子：
 
```python
import openai
 
client = openai.OpenAI(api_key="your-api-key")
 
# 定義您想要的函數結構
functions = [
    {
        "type": "function",
        "function": {
            "name": "get_product_info",
            "description": "獲取產品的詳細信息",
            "parameters": {
                "type": "object",
                "properties": {
                    "product_id": {
                        "type": "string",
                        "description": "產品的唯一標識符"
                    },
                    "name": {
                        "type": "string",
                        "description": "產品名稱"
                    },
                    "price": {
                        "type": "number",
                        "description": "產品價格"
                    },
                    "features": {
                        "type": "array",
                        "items": {
                            "type": "string"
                        },
                        "description": "產品特性列表"
                    },
                    "in_stock": {
                        "type": "boolean",
                        "description": "產品是否有庫存"
                    }
                },
                "required": ["product_id", "name", "price"]
            }
        }
    }
]
 
# 發送請求給 API
response = client.chat.completions.create(
    model="gpt-4-turbo",
    messages=[
        {"role": "system", "content": "您是一個產品信息助手。"},
        {"role": "user", "content": "請提供iPhone 15的產品信息"}
    ],
    functions=functions,
    function_call={"name": "get_product_info"}  # 強制調用特定函數
)
 
# 獲取函數調用結果
function_call = response.choices[0].message.function_call
function_name = function_call.name
function_args = function_call.arguments
 
print(f"函數名稱: {function_name}")
print(f"函數參數: {function_args}")
```
 
執行後會輸出類似以下的結構化 JSON 數據：
 
```
函數名稱: get_product_info
函數參數: {
  "product_id": "iphone15-base",
  "name": "iPhone 15",
  "price": 799.00,
  "features": ["A16 Bionic chip", "48MP camera system", "USB-C connector", "Dynamic Island"],
  "in_stock": true
}
```
 
重點說明：
1. 透過 `functions` 參數定義您需要的 JSON 結構
2. 使用 `function_call` 參數指定要調用的函數
3. 從響應中獲取結構化的 JSON 數據
4. 您可以根據需求設計任何符合 JSON Schema 規範的數據結構
 
這樣您就能獲得符合預期格式的 JSON 輸出，適合直接在應用程序中使用。
 
HKPC-DTD: n8n中怎麼用
 
GitHub Copilot: # 在 n8n 中使用 OpenAI Function Calling
 
在 n8n 工作流程中，您可以使用 OpenAI 節點來實現 Function Calling 功能。以下是具體步驟：
 
## 基本設置步驟
 
1. **添加 OpenAI 節點**:
   - 在 n8n 工作流中添加 "OpenAI" 節點
   - 如果您還沒有設置，需要創建 OpenAI 憑證
 
2. **配置 OpenAI 節點**:
   - 選擇操作: "Chat Completion"
   - 選擇模型: "gpt-4-turbo" 或其他支持 function calling 的模型
   - 添加系統消息和用戶消息
 
3. **設置 Function Calling**:
   - 開啟 "Functions" 選項
   - 添加您的自定義函數定義
 
## 具體示例
 
假設您要創建一個工作流，通過 Function Calling 獲取產品信息：
 
```javascript
// 在 OpenAI 節點的 Functions 字段中添加這段 JSON
[
  {
    "name": "get_product_info",
    "description": "獲取產品的詳細信息",
    "parameters": {
      "type": "object",
      "properties": {
        "product_id": {
          "type": "string",
          "description": "產品的唯一標識符"
        },
        "name": {
          "type": "string", 
          "description": "產品名稱"
        },
        "price": {
          "type": "number",
          "description": "產品價格"
        },
        "features": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "產品特性列表"
        },
        "in_stock": {
          "type": "boolean",
          "description": "產品是否有庫存"
        }
      },
      "required": ["product_id", "name", "price"]
    }
  }
]
```
 
然後在 "Function Call" 字段中設置:
```
get_product_info
```
 
## 完整工作流程示例
 
1. **HTTP Trigger 節點**: 接收請求，包含用戶查詢
 
2. **OpenAI 節點**: 配置如下
   - 操作: Chat Completion
   - 模型: gpt-4-turbo
   - 消息: 
     - 系統消息: "您是一個產品信息助手。"
     - 用戶消息: `{{$json.query}}` (從 HTTP 請求中獲取)
   - Functions: 如上述 JSON 所示
   - Function Call: "get_product_info"
 
3. **Code 節點**: 處理返回的 Function Call 結果
   ```javascript
   // 解析 OpenAI 返回的函數調用結果
   const functionCall = $input.all()[0].json.data.choices[0].message.function_call;
   const functionArgs = JSON.parse(functionCall.arguments);
   // 返回處理後的數據
   return {
     json: {
       productInfo: functionArgs
     }
   };
   ```
 
4. **Respond to Webhook 節點**: 返回處理後的數據
 
## 注意事項
 
1. 確保使用支持 Function Calling 的 OpenAI 模型
2. 正確設置 JSON 格式的函數定義
3. 您可以使用表達式 `{{$json.xxx}}` 動態設置消息內容
4. 在後續節點中處理返回的函數調用結果
 
這樣您就可以在 n8n 工作流中獲得結構化的 JSON 數據，用於後續處理或響應給用戶。
