# KẾ HOẠCH BẢN THẢO — TOKEN — Đường đi của mảnh tri thức

> Khóa hướng cho Chương 4–20 trước khi viết bản thảo đầy đủ.
>
> Đây là kế hoạch biên tập của cuốn sách, không phải hồ sơ lịch sử nghiên cứu.

## Luật chung cho mọi chương

Mỗi chương phải có:

1. một câu hỏi đời thường ở đầu;
2. sơ đồ nhỏ chỉ chứa kiến thức người đọc đã học;
3. không quá nhiều khái niệm mới cùng lúc;
4. tiếng Việt trước, thuật ngữ Anh trong ngoặc;
5. ít nhất một ví dụ cụ thể;
6. nếu dùng số liệu thực tế, phải ghi rõ số liệu chứng minh điều gì và không chứng minh điều gì;
7. một mục **Nhớ 3 điều**;
8. liên kết **Tiếp theo** tới chương kế;
9. không công bố tên công cụ nghiên cứu, kiến trúc sản phẩm, nhánh nghiên cứu hay lịch sử triển khai;
10. không biến tương quan, điểm nóng hay khuôn mẫu bộ đếm thành kết luận nhân quả.

---

## Chương 4 — Cùng một token, ngữ cảnh khác, biểu diễn khác

**Câu hỏi:** Vì sao cùng một token nhưng đứng trong hai câu khác nhau lại có thể dẫn tới trạng thái khác?

**Khái niệm mới:**
- ngữ cảnh (context);
- trạng thái ẩn (hidden state), chỉ giới thiệu ở mức “dãy số trung gian mô hình đang giữ”;
- biểu diễn theo ngữ cảnh (contextual representation).

**Không làm:** chưa dạy công thức attention.

---

## Chương 5 — Dòng tín hiệu đi xuyên Transformer

**Câu hỏi:** Nếu token đã thành một trạng thái số, nó đi qua những gì?

**Khái niệm:**
- lớp Transformer;
- chuẩn hóa;
- cơ chế chú ý;
- nhánh biến đổi tín hiệu;
- đường cộng tắt.

Chỉ giới thiệu vai trò. Chi tiết tách sang Chương 6–8.

---

## Chương 6 — Cơ chế chú ý: token nhìn những token khác như thế nào?

**Khái niệm mới:**
- truy vấn (query);
- khóa (key);
- giá trị (value);
- điểm chú ý;
- softmax.

**Cách dạy:** dùng ví dụ câu có đại từ hoặc từ đa nghĩa trước khi đưa Q/K/V.

**Ranh giới:** trọng số chú ý (attention weight) không phải lời giải thích hoàn chỉnh của “mô hình đang nghĩ gì”.

---

## Chương 7 — Nhánh biến đổi tín hiệu: điều gì xảy ra sau cơ chế chú ý?

**Khái niệm:**
- mạng truyền thẳng (feed-forward network, FFN);
- mở rộng chiều;
- phi tuyến;
- thu chiều trở lại.

Không tuyên bố FFN là “nơi chứa tri thức”.

---

## Chương 8 — Đường cộng tắt: vì sao thông tin cũ không biến mất hoàn toàn?

**Khái niệm:**
- đường cộng tắt (residual connection);
- dòng trạng thái tích lũy.

**Mục tiêu:** người đọc thấy trạng thái được cập nhật, không phải bị thay mới hoàn toàn mỗi lớp.

---

## Chương 9 — Một lớp chưa biết cả câu — vậy nhiều lớp làm được gì?

**Khái niệm:**
- sự kết hợp nhiều bước (composition);
- chiều sâu (depth);
- biến đổi dần qua nhiều lớp.

Không dùng câu “lớp thấp học cú pháp, lớp cao học ngữ nghĩa” như luật tuyệt đối.

---

## Chương 10 — Từ trạng thái cuối tới token tiếp theo

**Khái niệm:**
- lớp tạo điểm đầu ra;
- điểm dự đoán (logits);
- chọn token có điểm cao nhất;
- lấy mẫu (sampling) ở mức trực giác.

**Kết:** mở nửa sau — từ phép tính logic sang công việc vật lý.

---

## Chương 11 — Một phép tính logic không phải một chương trình GPU

**Khái niệm:**
- phép tính logic (semantic operation);
- chương trình GPU (kernel);
- lần giao việc (dispatch);
- công việc logic không đồng nhất với công việc vật lý.

**Mốc bằng chứng thực nghiệm:**
- 451 phép tính logic được gắn thời gian;
- 469 lần giao việc vật lý trong một bước sinh token.

**Giới hạn:** số liệu minh họa một hệ thống thật, không phải quy luật mọi LLM đều có 469 dispatch.

---

## Chương 12 — Một phép tính có thể trở thành nhiều lần giao việc cho phần cứng

**Ví dụ thực nghiệm:**
- lớp đầu ra: 1 phép tính logic / 19 lần giao việc vật lý.

**Khái niệm:**
- chia nhỏ công việc;
- hình học thực thi;
- phân rã vật lý.

---

## Chương 13 — Khi nhiều phép tính được gộp thành một công việc

**Khái niệm:**
- gộp phép tính (fusion);
- nhiều công việc logic có thể chia sẻ một lần giao việc vật lý.

**Mục tiêu:** phá giả định một phép tính = một chương trình GPU theo cả hai chiều.

---

## Chương 14 — Bộ nhớ cũng nằm trên đường đi của token

**Khái niệm:**
- đọc trọng số;
- dữ liệu trung gian;
- bộ nhớ đệm;
- DRAM;
- di chuyển dữ liệu và tính toán.

Không tuyên bố một điểm nóng bị giới hạn bởi bộ nhớ chỉ từ một khuôn mẫu bộ đếm định hướng.

---

## Chương 15 — Dấu vết thực thi: những dấu chân một token để lại

**Khái niệm:**
- dấu vết thực thi (execution trace);
- dấu thời gian (timestamp);
- đồng bộ;
- quy chiếu từ công việc vật lý về phép tính logic.

**Mốc bằng chứng thực nghiệm:**
- 469 lần giao việc vật lý;
- 469 lần giao việc có dấu thời gian;
- 451 phép tính logic được đo;
- không có định danh logic lạ;
- không có lần giao việc không quy chiếu được.

---

## Chương 16 — Thời gian nằm ở đâu?

**Mốc bằng chứng thực nghiệm:**
- lm_head ~63,91%;
- ffn_down ~16,27%;
- ffn_up ~6,01%;
- ffn_gate ~5,98%;
- top 4 ~92,17%.

**Khái niệm:**
- điểm nóng (hotspot);
- quy chiếu thời gian;
- định vị chi phí (localization).

**Câu khóa:**

> **Biết thời gian nằm ở đâu chưa có nghĩa biết vì sao nó nằm ở đó.**

---

## Chương 17 — Bộ đếm phần cứng: con số đo được có nghĩa gì?

**Khái niệm:**
- bộ đếm phần cứng (hardware counter);
- kênh đo (measurement channel);
- độ đầy đủ của phép đo (measurement adequacy).

**Mốc bằng chứng thực nghiệm:**
- 268 bộ đếm được trình điều khiển công bố;
- muốn lấy toàn bộ danh sách cần 12 lượt đo;
- một số bộ đếm quan trọng không dùng được;
- GpuTime = 0 không được diễn giải thành thời gian vật lý thật sự bằng 0.

**Câu khóa:**

> **Số đo bằng 0 không tự động nghĩa đại lượng vật lý bằng 0.**

---

## Chương 18 — Khi một khác biệt quá nhỏ vẫn đáng để hỏi

**Đây là chương gợi mở. Không tiết lộ bất kỳ chương trình nghiên cứu nội bộ nào.**

**Câu hỏi:**

~~~text
A ≈ B
nhưng
Δ = B - A ≠ 0

Δ là gì?
~~~

**Dạy:**
- sai khác (difference);
- phần dư (residual);
- độ phân giải (resolution);
- mức nhiễu (noise floor);
- khả năng lặp lại (repeatability);
- cấu trúc (structure).

**Chuỗi câu hỏi:**
- có lặp lại không?
- có phụ thuộc vị trí không?
- có thay đổi theo thời gian không?
- đổi đầu vào nhẹ thì phần dư có đổi có cấu trúc không?

**Hai câu khóa:**

> **Nhỏ không có nghĩa vô nghĩa.**

> **Có cấu trúc không có nghĩa đã biết nguyên nhân.**

Chương dừng ở câu hỏi.

---

## Chương 19 — Một khuôn mẫu đẹp vẫn chưa phải nguyên nhân

**Mốc bằng chứng thực nghiệm:**
- các chỉ số DRAM, LSC, SBID stall và ALU tạo ra sự tách biệt định hướng mạnh;
- kết quả chính thức vẫn KHÔNG ĐẠT / CHƯA GIẢI QUYẾT vì kênh đo chưa đủ tin cậy.

**Khái niệm:**
- tương quan;
- bằng chứng định hướng;
- kết luận nhân quả;
- biến gây nhiễu;
- can thiệp có kiểm soát.

**Bài học:**
khuôn mẫu mạnh vẫn không vượt qua được một kênh đo không đủ tin cậy.

---

## Chương 20 — Từ phép đo tới điều ta thật sự biết

**Khép hai đường của sách:**

~~~text
Bên trong mô hình:
token
↓
biểu diễn
↓
biến đổi
↓
đầu ra
~~~

và:

~~~text
Phía người quan sát:
hiện tượng
↓
phép đo
↓
bằng chứng
↓
diễn giải
↓
tri thức
~~~

**Kết luận cần tránh:** “tri thức nằm ở vectơ X / nút Y”.

**Kết luận phù hợp hơn:**

- token là điểm vào, không phải viên tri thức;
- trạng thái được tạo và biến đổi phụ thuộc cả trọng số và ngữ cảnh;
- thực thi vật lý là một lớp khác với phép tính logic;
- phép đo chỉ trở thành tri thức khi nguồn gốc, độ đầy đủ và ranh giới diễn giải được giữ rõ.

**Câu cuối dự kiến:**

> **Ta bắt đầu bằng cách đi theo một token để tìm “mảnh tri thức”. Cuối cùng, điều ta tìm thấy không phải một mảnh vật chất nằm yên, mà là một chuỗi biến đổi — và một kỷ luật để biết tới đâu mình thật sự hiểu chuỗi biến đổi đó.**
