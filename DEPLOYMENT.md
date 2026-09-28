# Thông Tin Deploy — Checkpoint 5

> Điền file này sau khi deploy xong. `pytest tests/test_cp5.py` đọc file này
> để tìm địa chỉ service của bạn và gọi thử.
>
> **Chỉ ghi TÊN biến môi trường, tuyệt đối không dán giá trị API key vào đây.**
> Repo này công khai — dán khóa vào là mất khóa.

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Phạm Thanh Trung |
| Mã học viên | 2A202602949 |
| Repo | https://github.com/trump22/K4-L3A-PhamThanhTrung-2A202602949-Cloud-Service-And-Deployment |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://day12-agent-unfc.onrender.com |
| Platform | Render Blueprint |
| Ngày deploy | 2026-09-28 |

## Biến Môi Trường Đã Set Trên Cloud

Ghi tên biến và **nguồn giá trị**, không ghi giá trị:

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | ✅ | Render tự gán |
| `AGENT_API_KEY` | ✅ | Nhập trong Render Dashboard; không lưu trong repo |
| `REDIS_URL` | ✅ | Tự nối tới Render Key Value `day12-redis` qua `render.yaml` |
| `RATE_LIMIT_PER_MINUTE` | ✅ | `10`, khai báo trong `render.yaml` |
| `MONTHLY_BUDGET_USD` | ✅ | `10.0`, khai báo trong `render.yaml` |
| `LOG_LEVEL` | ✅ | `INFO`, khai báo trong `render.yaml` |

## Lệnh Kiểm Tra

Public URL: `https://day12-agent-unfc.onrender.com`

```bash
# 1. Liveness — mong đợi 200 {"status":"ok"}
curl -i https://day12-agent-unfc.onrender.com/health

# 2. Readiness — mong đợi 200 {"status":"ready"} (đã nối được Redis)
curl -i https://day12-agent-unfc.onrender.com/ready

# 3. Không có API key — mong đợi 401
curl -i -X POST https://day12-agent-unfc.onrender.com/ask \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello"}'

# 4. Có API key — mong đợi 200 kèm câu trả lời
curl -i -X POST https://day12-agent-unfc.onrender.com/ask \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $AGENT_API_KEY" \
  -H "X-User-Id: sv-test" \
  -d '{"question":"Deploy là gì?"}'

# 5. Rate limit — gọi 15 lần, những lần cuối phải trả 429
for i in $(seq 1 15); do
  curl -s -o /dev/null -w "%{http_code} " -X POST https://day12-agent-unfc.onrender.com/ask \
    -H "Content-Type: application/json" \
    -H "X-API-Key: $AGENT_API_KEY" \
    -H "X-User-Id: sv-test" \
    -d '{"question":"test"}'
done; echo
```

## Kết Quả Chạy Thật

Dựa trên các lệnh HTTP chạy trực tiếp tới service Render ngày 2026-09-28:

```
/health: HTTP 200 OK — {"status":"ok","service":"day12-agent","version":"1.0.0"}
/ready: HTTP 200 OK — {"status":"ready","redis":true}
/ask không có API key: HTTP 401 Unauthorized — {"detail":"invalid or missing API key"}
/ask có API key: HTTP 200 OK — trả về answer, user_id, history_length, cost_usd và token usage (đã thử qua Render Swagger UI ngày 2026-09-28).
Rate limit: HTTP 429 Too Many Requests — {"detail":"rate limit exceeded"}; header `Retry-After: 60` (đã xác minh trên Render Swagger UI ngày 2026-09-28 với `X-User-Id: rate-test-01`).
Blueprint sync: thành công; đã tạo day12-agent và day12-redis.
```

## Ảnh Chụp Màn Hình

Ảnh minh chứng đã lưu trong `screenshots/`:

- `screenshots/dashboard.png` — Render Blueprint sync thành công
- `screenshots/health.png` — kết quả gọi `/health` trên Render
- `screenshots/ready.png` — kết quả gọi `/ready`, xác nhận Redis sẵn sàng (`redis: true`)
- `screenshots/ask-auth-200.png` — kết quả `POST /ask` có API key trả HTTP 200 qua Render Swagger UI; ảnh đã cắt bỏ vùng hiển thị key
- `screenshots/rate-limit-429.png` — kết quả `/ask` trả HTTP 429 và `Retry-After: 60`; ảnh đã cắt bỏ phần cURL có API key

---

## Nếu Dùng Phương Án Dự Phòng

Không đăng ký được tài khoản cloud? Vẫn nộp được bài, nhưng CP5 tối đa 60% điểm:

1. Đặt `LOCAL_FALLBACK=true` trong `.env`
2. Chạy `docker compose up -d` rồi kiểm tra `docker compose ps`
3. Chụp màn hình vào `screenshots/`
4. Chạy `pytest tests/test_cp5.py -v` — bộ test sẽ tự chuyển sang kiểm tra
   `http://localhost:8000`
5. Ghi rõ lý do không deploy được vào phần dưới đây:

Không áp dụng — đã triển khai trên Render.
