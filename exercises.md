# Phiếu Phản Ánh — K4 Level 3B, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng `> *Câu trả lời của bạn*` bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên:Nguyễn Đình Lâm Phúc  Mã học viên: 2A202602986

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Một tình huống cụ thể là khi deploy lên Render nhưng quên đặt AGENT_API_KEY trong Environment. Khi Settings được khởi tạo, thiếu khóa sẽ gây lỗi giúp tôi phát hiện và bổ sung trước khi service phục vụ người dùng. Nếu mặc định là "changeme", ứng dụng có thể tiếp tục chạy với khóa dễ đoán, khiến người khác gọi /ask trái phép.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> {"event": "ask_completed", "level": "info", "timestamp": "2026-09-29T04:39:31.595216+00:00", "user_id": "sv-test", "tokens_in": 46, "tokens_out": 50, "cost_usd": 3.69e-05}
Với log JSON của sự kiện ask_completed, tôi có thể:
 - Lọc theo user_id và cộng cost_usd để biết từng người dùng tiêu bao nhiêu.
 - Dựa vào timestamp và event để thống kê số lượt trả lời trong từng khoảng thời gian.
---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f <Dockerfile-1-stage> -t agent:single .
docker build -t agent:multi .
docker images | grep agent
```

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | 1024 MB |
| Multi-stage | 271 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Dockerfile mới dùng python:3.11-slim, chỉ copy dependency đã cài và source cần thiết sang runtime. So với bản đầu dùng Python image đầy đủ và COPY . ., phần giảm dung lượng có thể đến từ base image gọn hơn và việc loại bỏ file không cần thiết.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Nếu chỉ sửa app/main.py, bước COPY requirements.txt và RUN pip install ở builder vẫn có thể dùng cache vì dependency không đổi. Các bước runtime trước COPY app ./app cũng có thể dùng lại cache.
Layer COPY app ./app thay đổi; các bước phía sau cần được Docker đánh giá hoặc tạo lại. Nhờ vậy, thay đổi source không buộc phải cài lại thư viện.
Nếu đặt COPY . . trước RUN pip install, sửa source sẽ làm mất cache từ bước copy, khiến bước cài dependency phải chạy lại.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Nếu code Python có lỗ hổng cho phép thực thi lệnh, kẻ tấn công có thể chạy lệnh với quyền của tiến trình ứng dụng. Khi container chạy root, họ có quyền root bên trong container. Nếu tiếp tục khai thác được lỗ hổng thoát container hoặc cấu hình nguy hiểm như mount Docker socket, họ có thể giành quyền cao trên host.
Lệnh USER appuser khiến ứng dụng chạy bằng user thường, giảm quyền mà kẻ tấn công nhận được ngay sau khi khai thác ứng dụng. Nó giảm tác động của sự cố nhưng không bảo đảm ngăn mọi cách thoát container.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Với cách đếm theo phút đồng hồ, người dùng có thể gửi 20 request trong khoảng hai giây quanh thời điểm chuyển phút: 10 request ngay trước khi phút cũ kết thúc và 10 request ngay sau khi bộ đếm reset.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit giới hạn tần suất gọi, còn cost guard giới hạn chi phí tích lũy theo user và tháng.
Ví dụ: người dùng mới gọi một lần trong phút hiện tại, nhưng đã tiêu 11 USD trong tháng với ngân sách 10 USD. Request chưa vượt tần suất nhưng bị trả 402.
Ví dụ ngược lại: người dùng còn nhiều ngân sách, nhưng đã gọi 10 lần trong 60 giây. Lần thứ 11 bị rate limiter trả 429.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Nếu dùng chung một endpoint kiểm tra Redis cho cả liveness và readiness: 1, Redis mất kết nối. 2, Cả ba container kiểm tra Redis thất bại và trả 503 dù process vẫn hoạt động. 3, Load balancer ngừng gửi traffic đến các instance không ready. 4, Nếu lỗi kéo dài đủ ngưỡng liveness đã cấu hình, orchestrator có thể restart cả ba container. 5, Restart ứng dụng không sửa được Redis, nên service có thể tiếp tục thất bại và khởi động lại.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Khi chạy 3 replica và gọi /ask nhiều lần với cùng X-User-Id, history_length tăng tuần tự, ví dụ 0, 2, 4, 6, vì cả ba container cùng đọc và ghi lịch sử trong Redis. Nếu lịch sử được lưu trong một dict Python, mỗi container sẽ có bộ nhớ riêng. Khi request chuyển sang container khác, history_length có thể quay lại 0 hoặc một giá trị thấp hơn, khiến lịch sử bị mất hoặc không nhất quán.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Khi deploy lên Render, service không vượt qua health check vì ứng dụng thoát ngay lúc khởi động. Trong log có lỗi NotImplementedError: TODO (CP4): cài đặt install tại hàm lifecycle.install(). Tôi kiểm tra mục Logs của Render và thấy lỗi xảy ra trước khi Uvicorn bắt đầu lắng nghe cổng. Nguyên nhân là phần xử lý lifecycle của CP4 chưa được cài đặt. Tôi hoàn thiện hàm lifecycle.install(), build và deploy lại. Sau đó health check trả về 200 với {"status":"ok"} và service hoạt động bình thường.
