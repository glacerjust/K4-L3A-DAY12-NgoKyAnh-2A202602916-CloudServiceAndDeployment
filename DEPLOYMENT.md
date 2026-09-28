# Thông Tin Deploy — Checkpoint 5

> Điền file này sau khi deploy xong. `pytest tests/test_cp5.py` đọc file này
> để tìm địa chỉ service của bạn và gọi thử.
>
> **Chỉ ghi TÊN biến môi trường, tuyệt đối không dán giá trị API key vào đây.**
> Repo này công khai — dán khóa vào là mất khóa.

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Ngô Kỳ Anh |
| Mã học viên | 2A202602916 |
| Repo | https://github.com/glacerjust/K4-L3A-DAY12-NgoKyAnh-2A202602916-CloudServiceAndDeployment |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://day12-lab-production-a05f.up.railway.app |
| Platform | Railway |
| Ngày deploy | 2026-09-28 |

## Biến Môi Trường Đã Set Trên Cloud

Ghi tên biến và **nguồn giá trị**, không ghi giá trị:

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | ✅ | platform tự gán |
| `AGENT_API_KEY` | ✅ | đặt trong dashboard, không nằm trong repo |
| `REDIS_URL` | ✅ | Redis add-on của Railway |
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
1. Liveness:
HTTP/1.1 200 OK
Content-Type: application/json
Date: Mon, 28 Sep 2026 09:51:27 GMT
Server: railway-hikari
x-railway-request-id: C3JhCRlXTj-1n0sULPU1MQ
Content-Length: 57
x-hikari-trace: sin1.nzn2
x-railway-edge: sin1
Connection: keep-alive

{"status":"ok","service":"day12-agent","version":"1.0.0"}

2. Readiness:
HTTP/1.1 200 OK
Content-Type: application/json
Date: Mon, 28 Sep 2026 09:51:54 GMT
Server: railway-hikari
x-railway-request-id: 6KkJJXXRRLWVt5419I3ezw
Content-Length: 31
x-hikari-trace: sin1.98a6
x-railway-edge: sin1
Connection: keep-alive

{"status":"ready","redis":true}

3. Không có API key:
HTTP/1.1 401 Unauthorized
Content-Type: application/json
Date: Mon, 28 Sep 2026 09:52:30 GMT
Server: railway-hikari
x-railway-request-id: pmztzuMMQP-taBMR9I3ezw
Content-Length: 39
x-hikari-trace: sin1.hs0s
x-railway-edge: sin1
Connection: keep-alive

{"detail":"invalid or missing API key"}

4. Có API key:
{
    "answer":  "Ngáº¯n gá»n: Deploy la gi phá»¥ thuá»c vÃ o ba yáº¿u tá» â cáº¥u hÃ¬nh qua biáº¿n mÃ´i trÆ°á»ng, health check Äá» orchestrator biáº¿t tráº¡ng thÃ¡i, vÃ  giá»i háº¡n tÃ i nguyÃªn. (MÃ¬nh Äang nhá» 2 lÆ°á»£t trao Äá»i trÆ°á»c ÄÃ³.)",
    "user_id":  "sv-test",
    "history_length":  2,
    "cost_usd":  3.465E-05,
    "tokens":  {
                   "in":  43,
                   "out":  47
               }
}
5. Rate limit:
200 200 200 200 200 200 200 200 200 200 429 429 429 429 429 
```

## Ảnh Chụp Màn Hình

Đặt ảnh trong thư mục `screenshots/`:

- `screenshots/dashboard.png` — trang quản lý service trên platform
- `screenshots/health.png` — kết quả gọi `/health` từ trình duyệt hoặc curl
