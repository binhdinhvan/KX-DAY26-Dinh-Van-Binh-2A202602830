# Minh chứng thực hành Day 26

Ngày đối chiếu: 14/10/2026 (Asia/Saigon).

## Các tệp đã lưu

| Nội dung | Tệp | Xác nhận |
|---|---|---|
| Function Calling | [01-function-calling.png](01-function-calling.png) | Model yêu cầu tool, app chạy cho Hà Nội/Đà Nẵng và trả lời cuối |
| MCP Basics | [02-mcp-client.png](02-mcp-client.png) | `list_tools` và `call_tool` cho 3 thành phố |
| Authentication thành công | [03a-auth-ok.png](03a-auth-ok.png) | Token hợp lệ liệt kê và gọi được tool |
| Authentication thất bại | [03a-auth-fail.png](03a-auth-fail.png) | Client đặt wrong-token; server từ chối HTTP 401 |
| Registry | [03b-registry.png](03b-registry.png) | Tìm tag weather, chọn v2, gọi tool; tìm keyword forecast |
| Versioning | [03c-versioning.png](03c-versioning.png) | Metadata v2.0.0, deprecation, kết quả v1 và v2 có forecast |
| Kiểm tra setup ADK | [04-verify-setup.png](04-verify-setup.png) | All checks passed: cấu hình, dependency, cấu trúc, kết nối MCP, import agent |
| ADK thời tiết hiện tại | [04-adk-current.png](04-adk-current.png) | Agent gọi `get_current_weather` thành công, có dữ liệu và thời điểm cập nhật |
| ADK dự báo | [04-adk-forecast.png](04-adk-forecast.png) | Agent gọi `get_forecast`, trả đủ ngày 14–16/10/2026 |

Đã đủ bốn ảnh checkpoint Production và ba ảnh checkpoint ADK theo yêu cầu nộp bài được cung cấp. `health_check()` của Lab 04 cũng đã được gọi thành công qua MCP trong phiên kiểm tra; không có ảnh riêng cho tool này. Server Auth đã dừng trước khi dùng ADK ở cổng 8000.

## Chạy lại phần 03 trên Windows

Từ thư mục gốc, dùng môi trường `.venv` cài theo `requirements.txt` (MCP SDK v2). Lab 04 dùng môi trường riêng (MCP SDK v1).

Terminal 1:

```powershell
$env:PYTHONUTF8="1"
cd 03-production
..\.venv\Scripts\python.exe auth_server.py
```

Nếu cổng 8000 đã có Auth server đang chạy, dùng server đó. Terminal 2, bắt đầu tại thư mục gốc:

```powershell
$env:PYTHONUTF8="1"
cd 03-production
..\.venv\Scripts\python.exe auth_client.py
..\.venv\Scripts\python.exe registry_client.py
..\.venv\Scripts\python.exe versioned_client.py
```

Registry đã được sửa để đọc JSON bằng UTF-8 trên Windows. Registry có entry email/database để minh họa; các server nội bộ đó không được triển khai hoặc gọi trong bài thực hành này.

## Chạy lại Lab 04

Mỗi thư mục server/client có môi trường riêng được cài từ `pyproject.toml`. Terminal server, bắt đầu tại thư mục gốc:

```powershell
cd 04-lab/mcp-server
$env:WEATHERAPI_KEY="<khóa WeatherAPI của bạn>"
$env:PORT="8085"
.\.venv\Scripts\python.exe weather.py
```

Server hiện đọc khóa từ biến môi trường; tự tạo file `.env` chưa làm server tự nạp khóa khi chạy lệnh trên. Không đưa giá trị khóa vào ảnh hoặc commit.

Terminal client, bắt đầu tại thư mục gốc:

```powershell
cd 04-lab/mcp-client
# .env chứa GOOGLE_API_KEY và MCP_SERVER_URL=http://localhost:8085/mcp
.\.venv\Scripts\adk.exe web --port 8001
```

Mở `http://localhost:8001`, chọn `weather_agent`, hỏi thời tiết hiện tại và dự báo 3 ngày cho Hà Nội. Forecast của API trong lần chạy gồm hôm nay và hai ngày tiếp theo. Cloud Run chưa được triển khai.
