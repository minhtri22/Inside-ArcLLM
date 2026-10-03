# Chương 14 — Bộ nhớ cũng nằm trên đường đi của token

> **Mức đọc: Đi sâu**
>
> **Ta đã nói nhiều về tính toán:**
>
> ~~~text
> phép tính logic
> ↓
> kernel
> ↓
> dispatch
> ↓
> GPU tính
> ~~~
>
> Nhưng còn thiếu một nửa rất lớn:
>
> ~~~text
> dữ liệu phải có mặt
> đúng nơi
> đúng lúc
> ~~~

Hãy tưởng tượng một đầu bếp có thể thái rau cực nhanh.

Nhưng nguyên liệu đang ở kho cách bếp 100 mét.

Mỗi lần cần cà rốt, đầu bếp phải đứng chờ người mang cà rốt tới.

Khi đó tốc độ con dao không còn là toàn bộ câu chuyện.

GPU cũng vậy.

## Tính toán cần dữ liệu

Một phép nhân ma trận cần:

- trọng số;
- trạng thái đầu vào;
- vùng để ghi kết quả;
- đôi khi thêm các hệ số hoặc dữ liệu phụ.

Ta có thể vẽ:

~~~text
trọng số ───────┐
                │
trạng thái ─────┼→ phép tính → kết quả
                │
dữ liệu phụ ────┘
~~~

Nếu dữ liệu chưa tới, đơn vị tính toán có thể phải chờ.

## Bộ nhớ không chỉ có một tầng

Máy tính có nhiều nơi giữ dữ liệu.

Ở mức rất đơn giản:

~~~text
ổ lưu trữ
   ↓
RAM hệ thống
   ↓
vùng bộ nhớ GPU có thể truy cập
   ↓
bộ nhớ đệm
   ↓
thanh ghi / vùng rất gần đơn vị tính toán
~~~

Không phải mọi máy đều có ranh giới vật lý giống hệt sơ đồ này.

Ví dụ GPU tích hợp có thể dùng chung bộ nhớ vật lý với CPU.

Nhưng trực giác vẫn đúng:

> **Dữ liệu càng gần nơi tính toán và càng được tái sử dụng tốt, chi phí lấy dữ liệu có thể càng thấp.**

## Trọng số là phần rất lớn của đường đi

Khi sinh một token, mô hình phải dùng rất nhiều trọng số đã học.

Những trọng số đó không tự nằm sẵn trong đơn vị nhân ma trận.

Chúng phải được đọc từ đâu đó.

Nếu một khối trọng số lớn được dùng cho mỗi token nhưng tái sử dụng kém, việc đọc dữ liệu có thể chiếm phần đáng kể thời gian.

Đây là lý do người ta quan tâm tới:

- băng thông bộ nhớ;
- bộ nhớ đệm;
- cách đóng gói trọng số;
- lượng tử hóa;
- cách chia công việc.

## Dữ liệu trung gian cũng phải đi đâu đó

Không chỉ trọng số.

Khi attention hay FFN tạo ra trạng thái mới, kết quả đó cũng phải được giữ để bước sau dùng.

Ta có:

~~~text
phép tính A
   ↓
dữ liệu trung gian
   ↓
phép tính B
~~~

Nếu A và B tách biệt:

~~~text
A
↓
ghi dữ liệu
↓
bộ nhớ
↓
đọc lại
↓
B
~~~

Nếu hệ thống gộp tốt, có thể giảm một phần vòng đi-về này.

Đây là lý do Chương 13 và Chương 14 nối với nhau.

## Bộ nhớ đệm là gì?

**Bộ nhớ đệm (cache)** là vùng nhỏ hơn, nhanh hơn, cố gắng giữ lại dữ liệu có khả năng sớm được dùng lại.

Có thể hình dung như bàn làm việc.

Kho lớn chứa tất cả tài liệu.

Nhưng những tờ đang dùng được đặt ngay trên bàn.

~~~text
kho tài liệu
    ↓
bàn làm việc
    ↓
tay đang xử lý
~~~

Nếu tài liệu cần dùng đã có trên bàn, ta không phải chạy về kho.

Trong phần cứng:

~~~text
bộ nhớ lớn
    ↓
cache
    ↓
đơn vị tính toán
~~~

## Tỷ lệ trúng cache

Khi dữ liệu cần dùng đã có trong cache, ta gọi đó là một **lần trúng bộ nhớ đệm (cache hit)**.

Tỷ lệ trúng cao có thể là tín hiệu rằng dữ liệu đang được tái sử dụng tốt hơn.

Nhưng đây là nơi phải bắt đầu cẩn thận.

Một tỷ lệ cache hit thấp **không tự động chứng minh**:

> “Kernel này chậm vì bộ nhớ.”

Có thể còn nhiều yếu tố khác.

## Bị giới hạn bởi tính toán hay bởi bộ nhớ?

Bạn có thể gặp hai cách nói:

- **bị giới hạn bởi tính toán (compute-bound)**;
- **bị giới hạn bởi bộ nhớ (memory-bound)**.

Ở mức trực giác:

~~~text
compute-bound
→ phần tính toán là giới hạn chính

memory-bound
→ việc cung cấp / di chuyển dữ liệu là giới hạn chính
~~~

Nhưng để gắn một nhãn như vậy cho một bài đo thật, ta cần bằng chứng phù hợp.

Không nên nhìn một bộ đếm rồi phán ngay.

## Một khuôn mẫu rất hấp dẫn

Trong một phép đo phần cứng thật, một nhóm phép tính có:

- mức khuếch đại đọc DRAM cao hơn nhóm so sánh;
- tỷ lệ trúng cache thấp hơn;
- mức chờ liên quan tới phụ thuộc dữ liệu cao hơn;
- mức sử dụng một số đơn vị tính toán thấp hơn.

Nhìn vào đó, câu chuyện:

> “Đây chắc chắn là vấn đề bộ nhớ.”

rất hấp dẫn.

Nhưng kết luận khoa học cuối cùng của phép đo đó **không** cho phép nói như vậy.

Lý do là một số bộ đếm quan trọng không đủ đáng tin để phân biệt cơ chế.

Ta sẽ mở toàn bộ câu chuyện ở Chương 17 và 19.

Ở đây chỉ cần học một nguyên tắc:

> **Một khuôn mẫu phù hợp với giả thuyết bộ nhớ chưa phải bằng chứng nhân quả rằng bộ nhớ là nguyên nhân.**

## Đường đi của token bây giờ có thêm một lớp

Nửa đầu sách:

~~~text
token
↓
biểu diễn
↓
attention / FFN
↓
nhiều lớp
↓
logits
~~~

Bây giờ thêm lớp vật lý:

~~~text
phép tính logic
↓
dispatch
↓
đọc trọng số
↓
đọc trạng thái
↓
tính toán
↓
ghi kết quả
↓
phép tính tiếp theo
~~~

Token không phải một hạt vật chất chạy qua dây dẫn.

Nhưng **công việc phục vụ trạng thái của token** tạo ra một chuỗi hoạt động vật lý rất thật trên máy.

## Làm sao nhìn thấy chuỗi đó?

Ta cần ghi lại:

- công việc nào đã chạy;
- khi nào bắt đầu;
- khi nào kết thúc;
- nó thuộc phép tính logic nào;
- có điểm đồng bộ nào giữa chúng.

Một bản ghi như vậy được gọi là **dấu vết thực thi (execution trace)**.

Đó là Chương 15.

### Nhớ 3 điều

1. **Tốc độ tính toán không phải toàn bộ câu chuyện; trọng số và dữ liệu trung gian phải được đưa tới nơi tính toán đúng lúc.**
2. **Bộ nhớ đệm (cache) có thể giảm chi phí lấy lại dữ liệu, nhưng một chỉ số cache đơn lẻ chưa đủ để kết luận cơ chế gây chậm.**
3. **Đường đi vật lý của một bước sinh token gồm cả tính toán lẫn di chuyển dữ liệu; muốn quan sát nó, ta cần một dấu vết thực thi.**

**Tiếp theo: [Chương 15 — Dấu vết thực thi: những dấu chân một token để lại](15-dau-vet-thuc-thi.md)**
