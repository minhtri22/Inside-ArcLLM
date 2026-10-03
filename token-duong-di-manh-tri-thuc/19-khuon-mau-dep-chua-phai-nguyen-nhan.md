# Chương 19 — Một khuôn mẫu đẹp vẫn chưa phải nguyên nhân

> **Mức đọc: Nâng cao**
>
> **Ta đang đứng trước một cám dỗ rất lớn:**
>
> ~~~text
> hai nhóm khác nhau rất rõ
>          ↓
> "chắc đây là nguyên nhân"
> ~~~
>
> Chương này giải thích vì sao một khuôn mẫu mạnh có thể là **bằng chứng định hướng**, nhưng vẫn chưa đủ để trở thành **kết luận nhân quả**.

Giả sử ta quan sát hai nhóm phép tính.

Nhóm A có:

- đọc bộ nhớ nhiều hơn;
- tỷ lệ trúng bộ nhớ đệm thấp hơn;
- thời gian chờ cao hơn;
- mức sử dụng một số đơn vị tính toán thấp hơn.

Nhóm B thì ngược lại.

Một câu chuyện rất tự nhiên xuất hiện:

> “A chậm vì đang bị nghẽn bộ nhớ.”

Có thể đúng.

Nhưng “có thể đúng” khác “đã chứng minh”.

## Tương quan là gì?

Khi hai đại lượng thay đổi cùng nhau, ta có thể nói chúng có **tương quan (correlation)**.

Ví dụ:

~~~text
nhiệt độ tăng
↓
doanh số kem tăng
~~~

Hai thứ có quan hệ.

Nhưng không phải vì kem làm thời tiết nóng hơn.

Một yếu tố khác — mùa hè — ảnh hưởng cả hai.

Yếu tố làm ta dễ nhầm quan hệ như vậy được gọi là **biến gây nhiễu (confounder)**.

Trong hệ thống máy tính cũng vậy.

## Một khuôn mẫu thật rất thuyết phục

Trong một bài đo cụ thể, khi so một nhóm LM-head Q6 với FFN-down Q6 ở một bài đo đầu vào ngắn, các quan sát định hướng gồm:

~~~text
Khuếch đại đọc DRAM
LM head      4,268
FFN-down     1,436

Tỷ lệ trúng LSC
LM head      0,103
FFN-down     0,738

XVE SBID stall
LM head      81,78%
FFN-down     56,50%

Mức sử dụng ALU1
LM head      ~1,58%
FFN-down     ~13,84%
~~~

Chỉ nhìn bốn dòng này, một câu chuyện rất dễ viết:

~~~text
đọc DRAM nhiều
+
cache kém
+
stall cao
+
ALU thấp
↓
"nghẽn bộ nhớ"
~~~

Nhưng nghiên cứu không được phép dừng ở câu chuyện đẹp nhất.

## Vì sao khuôn mẫu đó chưa đủ?

Bởi những bộ đếm cần thiết để phân biệt cơ chế không phải tất cả đều đáng tin.

Trong cùng chương trình đo:

- một số bộ đếm quan trọng không dùng được;
- GpuTime trả 0;
- XVE_STALL có nhóm đo trả 0 nhưng nhóm khác trả khác 0;
- GPU_MEMORY_REQUEST_QUEUE_FULL cũng bất nhất giữa các nhóm.

Do đó, dù khuôn mẫu mô tả rất mạnh, **độ đầy đủ của kênh đo không đạt** cho câu hỏi xác nhận cơ chế.

Kết luận chính thức phải là:

~~~text
hỗ trợ cơ chế chính thức: KHÔNG CÓ

kết luận nhân quả: KHÔNG CÓ

trạng thái:
KHÔNG ĐẠT / CHƯA GIẢI QUYẾT
~~~

Đây không phải thất bại của khoa học.

Đây chính là khoa học hoạt động đúng.

## Bằng chứng định hướng

Ta không cần vứt bỏ các con số.

Chúng vẫn có thể được giữ như **bằng chứng định hướng (directional evidence)**.

Nghĩa là:

> Chúng chỉ ra một hướng đáng kiểm tra tiếp.

Không phải:

> Chúng đã chứng minh hướng đó đúng.

Ta có thể vẽ:

~~~text
khuôn mẫu
   ↓
giả thuyết hợp lý
   ↓
thiết kế phép thử phân biệt
   ↓
bằng chứng đủ mạnh
   ↓
mới cân nhắc kết luận
~~~

Nếu dừng ở bước đầu mà viết luôn bước cuối, ta đã nhảy qua phần khó nhất.

## “Vì sao” cần mạnh hơn “ở đâu”

Chương 16 đã nói:

~~~text
định vị
→ thời gian nằm ở đâu?
~~~

Chương 19 hỏi:

~~~text
cơ chế
→ vì sao lại tốn thời gian ở đó?
~~~

Câu hỏi thứ hai khó hơn.

Để nói “vì sao”, ta thường cần một phép thử có khả năng phân biệt giữa các giải thích cạnh tranh.

Ví dụ:

~~~text
Giả thuyết A:
chậm vì đọc bộ nhớ

Giả thuyết B:
chậm vì hình học thực thi

Giả thuyết C:
chậm vì phụ thuộc tuần tự
~~~

Một phép đo tốt phải giúp ta loại hoặc hỗ trợ các giả thuyết này theo cách đã định trước.

## Can thiệp có kiểm soát

Một cách mạnh để học về nguyên nhân là **can thiệp (intervention)**.

Thay vì chỉ nhìn hệ thống đang chạy, ta thay đổi có kiểm soát một yếu tố.

Ví dụ tưởng tượng:

~~~text
giữ dữ liệu giống nhau
giữ thuật toán giống nhau
chỉ đổi cách chia công việc
↓
đo phản ứng
~~~

Nếu hiệu ứng thay đổi đúng như dự đoán, bằng chứng nhân quả mạnh hơn.

Ta có thể vẽ:

~~~text
trạng thái ban đầu
       ↓
thay đổi đúng 1 yếu tố
       ↓
quan sát phản ứng
       ↓
so với đối chứng
       ↓
đánh giá giả thuyết
~~~

Nhưng một can thiệp cũng phải giữ tính đúng và tránh thay đổi nhiều yếu tố cùng lúc.

## Can thiệp không phải phép màu

Ngay cả khi đổi một kernel và thấy nhanh hơn, ta vẫn phải hỏi:

- đầu ra còn đúng không?
- có thay nhiều thứ cùng lúc không?
- phép đo có ổn định không?
- lợi ích có đi lên toàn hệ không?
- kết quả có lặp lại không?

Nhân quả không đến từ từ “intervention”.

Nó đến từ **thiết kế phép thử đủ phân biệt**.

## Một sơ đồ kỷ luật bằng chứng

Có thể tóm tắt cả phần cuối sách bằng:

~~~text
QUAN SÁT
   ↓
KHUÔN MẪU
   ↓
GIẢ THUYẾT
   ↓
PHÉP ĐO ĐỦ?
   ├─ không → CHƯA GIẢI QUYẾT
   │
   └─ có
       ↓
PHÉP THỬ PHÂN BIỆT / CAN THIỆP
       ↓
KẾT QUẢ
       ↓
KẾT LUẬN TRONG ĐÚNG PHẠM VI
~~~

“CHƯA GIẢI QUYẾT” không phải ô trống.

Nó là một trạng thái tri thức:

> **Ta biết bằng chứng hiện tại chưa đủ.**

## Quay lại “mảnh tri thức”

Điều thú vị là kỷ luật này không chỉ dành cho bộ đếm hiệu năng.

Nó cũng áp dụng khi ta nhìn biểu diễn trong mô hình.

Giả sử ta thấy một hướng trong không gian trạng thái liên quan mạnh với:

~~~text
thủ đô
~~~

Ta chưa được phép nói ngay:

> “Đây là chiều lưu tri thức thủ đô.”

Ta có thể nói:

> “Có một tín hiệu liên quan tới thuộc tính đang đo.”

Muốn nói mạnh hơn, cần các phép thử mạnh hơn.

Cùng một nguyên tắc chạy xuyên cả cuốn sách:

> **Tín hiệu không tự động trở thành cơ chế.**

## Còn câu hỏi cuối cùng

Ta đã đi từ:

~~~text
văn bản
↓
token
↓
biểu diễn
↓
nhiều lớp
↓
công việc vật lý
↓
dấu vết
↓
bộ đếm
↓
khuôn mẫu
~~~

Nhưng tên sách là:

> **Đường đi của mảnh tri thức**

Vậy sau toàn bộ hành trình này, “tri thức” nằm ở đâu?

Câu trả lời cần được viết cẩn thận hơn rất nhiều so với lúc bắt đầu.

Đó là Chương 20.

### Nhớ 3 điều

1. **Một khuôn mẫu định hướng mạnh vẫn chưa phải bằng chứng nhân quả.** Trong ví dụ thật, nhiều bộ đếm tách biệt rõ nhưng kết luận cơ chế vẫn là KHÔNG ĐẠT / CHƯA GIẢI QUYẾT vì kênh đo không đủ.
2. **Bằng chứng định hướng giúp tạo giả thuyết; muốn nói “vì sao” cần phép thử có khả năng phân biệt các giải thích cạnh tranh, thường mạnh hơn khi có can thiệp có kiểm soát.**
3. **CHƯA GIẢI QUYẾT là một kết quả khoa học có giá trị.** Nó nói rõ ranh giới của điều ta đang biết thay vì lấp chỗ trống bằng câu chuyện hợp lý nhất.

**Tiếp theo: [Chương 20 — Từ phép đo tới điều ta thật sự biết](20-tu-phep-do-toi-dieu-ta-biet.md)**
