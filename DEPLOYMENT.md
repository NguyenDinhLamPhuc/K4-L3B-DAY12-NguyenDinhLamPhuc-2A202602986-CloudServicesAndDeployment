# Thông Tin Deploy — Checkpoint 5

> Điền file này sau khi deploy xong. `pytest tests/test_cp5.py` đọc file này
> để tìm địa chỉ service của bạn và gọi thử.
>
> **Chỉ ghi TÊN biến môi trường, tuyệt đối không dán giá trị API key vào đây.**
> Repo này công khai — dán khóa vào là mất khóa.

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Nguyễn Đình Lâm Phúc |
| Mã học viên | 2A202602986 |
| Repo | https://github.com/NguyenDinhLamPhuc/K4-L3B-DAY12-NguyenDinhLamPhuc-2A202602986-CloudServicesAndDeployment |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://day12-agent-7xwc.onrender.com |
| Platform | Render |
| Ngày deploy | 29/9/2026 |

## Biến Môi Trường Đã Set Trên Cloud

Ghi tên biến và **nguồn giá trị**, không ghi giá trị:

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | ✅ | platform tự gán |
| `AGENT_API_KEY` | ✅ | đặt trong dashboard, không nằm trong repo |
| `REDIS_URL` | ✅ | redis://red-datjgaid0e5s73cc2b90:6379 |
| `RATE_LIMIT_PER_MINUTE` | ✅ | 10 |
| `MONTHLY_BUDGET_USD` | ✅ | 10.0 |
| `LOG_LEVEL` | ✅ | INFO |

## Lệnh Kiểm Tra

Thay `<URL>` bằng Public URL ở trên:

```bash
# 1. Liveness — mong đợi 200 {"status":"ok"}
curl -i <URL>/health

# 2. Readiness — mong đợi 200 {"status":"ready"} (đã nối được Redis)
curl -i <URL>/ready

# 3. Không có API key — mong đợi 401
curl -i -X POST <URL>/ask \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello"}'

# 4. Có API key — mong đợi 200 kèm câu trả lời
curl -i -X POST <URL>/ask \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $AGENT_API_KEY" \
  -H "X-User-Id: sv-test" \
  -d '{"question":"Deploy là gì?"}'

# 5. Rate limit — gọi 15 lần, những lần cuối phải trả 429
for i in $(seq 1 15); do
  curl -s -o /dev/null -w "%{http_code} " -X POST <URL>/ask \
    -H "Content-Type: application/json" \
    -H "X-API-Key: $AGENT_API_KEY" \
    -H "X-User-Id: sv-test" \
    -d '{"question":"test"}'
done; echo
```

## Kết Quả Chạy Thật

Dán output của các lệnh trên vào đây:

```
HTTP/1.1 200 OK
Date: Tue, 29 Sep 2026 04:14:59 GMT
Content-Type: application/json
Transfer-Encoding: chunked
Connection: keep-alive
cf-cache-status: DYNAMIC
rndr-id: a1ba8738-43db-4100
Server: cloudflare
vary: Accept-Encoding
x-render-origin-server: uvicorn
CF-RAY: a4282a07ab6da1ac-SIN
alt-svc: h3=":443"; ma=86400

{"status":"ok","service":"day12-agent","version":"1.0.0"}HTTP/1.1 200 OK
Date: Tue, 29 Sep 2026 04:15:00 GMT
Content-Type: application/json
Transfer-Encoding: chunked
Connection: keep-alive
cf-cache-status: DYNAMIC
rndr-id: 07b3203c-fa68-428d
Server: cloudflare
vary: Accept-Encoding
x-render-origin-server: uvicorn
CF-RAY: a4282a0aafe85fb1-SIN
alt-svc: h3=":443"; ma=86400

{"status":"ready","redis":true}HTTP/1.1 401 Unauthorized
Date: Tue, 29 Sep 2026 04:15:01 GMT
Content-Type: application/json
Transfer-Encoding: chunked
Connection: keep-alive
cf-cache-status: DYNAMIC
rndr-id: 26f69e1f-7b91-4204
Server: cloudflare
vary: Accept-Encoding
x-render-origin-server: uvicorn
CF-RAY: a4282a0e2c6fd25b-SIN
alt-svc: h3=":443"; ma=86400

{"detail":"invalid or missing API key"}HTTP/1.1 200 OK
Date: Tue, 29 Sep 2026 04:15:01 GMT
Content-Type: application/json
Transfer-Encoding: chunked
Connection: keep-alive
cf-cache-status: DYNAMIC
rndr-id: 9315fe35-4bc8-4a73
Server: cloudflare
vary: Accept-Encoding
x-render-origin-server: uvicorn
CF-RAY: a4282a10dd526bed-SIN
alt-svc: h3=":443"; ma=86400

{"answer":"Câu hỏi hay. Deploy là gì thường được giải quyết bằng cách chuẩn hóa môi trường chạy: cùng một image chạy giống nhau ở laptop và trên cloud.","user_id":"sv-test","history_length":0,"cost_usd":2.145e-05,"tokens":{"in":3,"out":35}}Lần 1: HTTP 200
Lần 2: HTTP 200
Lần 3: HTTP 200
Lần 4: HTTP 200
Lần 5: HTTP 200
Lần 6: HTTP 200
Lần 7: HTTP 200
Lần 8: HTTP 200
Lần 9: HTTP 200
Lần 10: HTTP 200
Lần 11: HTTP 429
Lần 12: HTTP 429
Lần 13: HTTP 429
Lần 14: HTTP 429
Lần 15: HTTP 429
```

## Ảnh Chụp Màn Hình

Đặt ảnh trong thư mục `screenshots/`:

- `screenshots/dashboard.png` — trang quản lý service trên platform
- `screenshots/health.png` — kết quả gọi `/health` từ trình duyệt hoặc curl

---

