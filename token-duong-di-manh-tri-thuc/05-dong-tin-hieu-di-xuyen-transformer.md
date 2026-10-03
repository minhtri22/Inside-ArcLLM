# Chương 5 — Dòng tín hiệu đi xuyên Transformer

> **Mức đọc: Đi sâu**
>
> **Ta đã đi tới đâu?**
>
> ~~~text
> token
>   ↓
> biểu diễn ban đầu
>   ↓
> trạng thái phụ thuộc ngữ cảnh
>   ↓
> [ nhiều lớp Transformer (layers) ]
> ~~~
>
> Chương này chưa mở công thức. Mục tiêu chỉ là nhìn được **bản đồ bên trong một lớp** trước khi đi sâu vào từng phần.

Ở Chương 4, ta liên tục nói rằng trạng thái của token được cập nhật khi đi qua nhiều lớp.

Nhưng **một lớp xử lý (layer)** là gì?

Ta thử mở một chiếc hộp.

## Một lớp không phải một phép tính duy nhất

Hãy hình dung một dây chuyền.

Một vật đi vào.

Nó trải qua vài trạm xử lý.

Sau đó đi ra với trạng thái đã thay đổi.

Một lớp Transformer (Transformer layer) cũng có thể được nhìn theo cách gần như vậy.

Với loại mô hình ngôn ngữ sinh token phổ biến hiện nay, một lớp thường có những thành phần chính như:

- bước chuẩn hóa;
- cơ chế cho các vị trí lấy thông tin từ nhau;
- một nhánh biến đổi trạng thái;
- các đường cộng lại tín hiệu cũ.

Tên kỹ thuật của chúng sẽ xuất hiện ngay sau khi ta có hình dung.

## Bản đồ đơn giản của một lớp

Ta bắt đầu từ trạng thái của các token:

~~~text
trạng thái đi vào lớp
        │
        ↓
   [ chuẩn hóa ]
        │
        ↓
 [ cơ chế chú ý ]
        │
        ↓
   cộng với đường cũ
        │
        ↓
   [ chuẩn hóa ]
        │
        ↓
 [ nhánh biến đổi ]
        │
        ↓
   cộng với đường cũ
        │
        ↓
trạng thái đi ra khỏi lớp
~~~

Đây là bản đồ đủ tốt để đọc các chương tiếp theo.

Các kiến trúc Transformer cụ thể có thể khác nhau về chi tiết, thứ tự hoặc công thức.

Cuốn sách này chủ yếu dùng trực giác của mô hình ngôn ngữ kiểu **chỉ dùng bộ giải mã (decoder-only)**, vì đây là dạng rất phổ biến trong các LLM sinh văn bản.

## Chuẩn hóa để làm gì?

Trạng thái bên trong mô hình là những dãy số.

Sau nhiều phép biến đổi, độ lớn và phân bố của các con số có thể thay đổi.

Một bước **chuẩn hóa (normalization)** giúp đưa tín hiệu về một dạng thuận lợi hơn cho phép tính tiếp theo.

Một loại thường gặp là **chuẩn hóa RMS (RMSNorm)**.

Ta chưa cần công thức.

Chỉ cần hình dung:

~~~text
dãy số đầu vào
      ↓
điều chỉnh thang đo
      ↓
dãy số phù hợp hơn cho bước tiếp
~~~

Chuẩn hóa không phải phần “mang tri thức” theo nghĩa một chiếc hộp chứa thông tin.

Nó là một phần của cách dòng tín hiệu được giữ trong vùng hoạt động phù hợp.

## Cơ chế chú ý: nhìn sang những vị trí khác

Đây là phần nổi tiếng nhất của Transformer.

Tên của nó là **cơ chế chú ý (attention)**.

Hãy trở lại câu:

~~~text
Tôi nhặt một hòn đá.
~~~

Khi xử lý vị trí “đá”, trạng thái hiện tại có thể cần thông tin từ những vị trí trước như “nhặt”, “một”, “hòn”.

Cơ chế chú ý tạo ra một cách để một vị trí tính xem:

> Những vị trí nào trước đó có liên quan tới việc cập nhật trạng thái của tôi?

Ta có thể hình dung:

~~~text
[Tôi] [nhặt] [một] [hòn] [đá]
  │      │      │      │     ↑
  └──────┴──────┴──────┴─────┘
        thông tin có thể
        được gom về đây
~~~

Đây chỉ là trực giác.

Chương 6 sẽ mở bên trong và giới thiệu truy vấn, khóa, giá trị và điểm chú ý.

## Nhánh biến đổi: xử lý trạng thái tại từng vị trí

Sau khi một vị trí đã lấy thêm thông tin liên quan từ chuỗi, trạng thái của nó còn đi qua một nhánh biến đổi khác.

Nhánh này thường được gọi là **mạng truyền thẳng (Feed-Forward Network, FFN)**.

Ở mức trực giác:

~~~text
trạng thái hiện tại
      ↓
mở rộng sang không gian lớn hơn
      ↓
biến đổi phi tuyến
      ↓
thu về kích thước cần dùng
      ↓
trạng thái mới
~~~

Nếu attention chủ yếu giúp **trao đổi thông tin giữa các vị trí**, FFN có thể được hình dung như một trạm **biến đổi trạng thái tại từng vị trí**.

Đây cũng chỉ là một **mô hình tinh thần (mental model)**.

Chương 7 sẽ mở nó kỹ hơn.

## Đường cộng tắt: không vứt bỏ trạng thái cũ

Có một chi tiết rất quan trọng trong sơ đồ lớp:

~~~text
trạng thái cũ ───────────────┐
                             │
             phép biến đổi   │
                  ↓          │
             kết quả mới     │
                  │          │
                  └──── cộng ┘
                        ↓
                  trạng thái sau
~~~

Đường đi tắt đó được gọi là **đường cộng tắt (residual connection)**.

Nó cho phép phần tín hiệu cũ đi vòng qua một phép biến đổi rồi được cộng lại với phần thay đổi mới.

Nhờ vậy, lớp không nhất thiết phải “xóa sạch rồi viết lại” toàn bộ trạng thái.

Chương 8 sẽ dành riêng cho ý này.

## Một lớp chỉ là một lần cập nhật

Nếu có nhiều lớp:

~~~text
biểu diễn ban đầu
       ↓
     lớp 1
       ↓
 trạng thái 1
       ↓
     lớp 2
       ↓
 trạng thái 2
       ↓
     lớp 3
       ↓
 trạng thái 3
       ↓
      ...
~~~

mỗi lớp có cơ hội:

- lấy thêm thông tin từ ngữ cảnh;
- biến đổi trạng thái;
- giữ lại và cộng thêm những thay đổi.

Điều quan trọng là:

> **Không có một lớp duy nhất phải làm toàn bộ công việc.**

Một lớp có thể tạo ra những thay đổi mà lớp sau tiếp tục sử dụng.

Đó là lý do chiều sâu của mô hình trở thành một phần quan trọng của câu chuyện.

## Một sơ đồ đầy đủ hơn một chút

Bây giờ ta có thể ghép lại:

~~~text
                 MỘT LỚP TRANSFORMER

trạng thái vào
     │
     ├─────────────── đường cũ ───────────────┐
     ↓                                        │
chuẩn hóa                                     │
     ↓                                        │
cơ chế chú ý                                  │
     ↓                                        │
kết quả mới ──────────────────────────────── cộng
                                              ↓
                                      trạng thái giữa
                                              │
                                              ├──── đường cũ ────┐
                                              ↓                   │
                                         chuẩn hóa                │
                                              ↓                   │
                                      mạng truyền thẳng           │
                                              ↓                   │
                                       kết quả mới ──────────── cộng
                                                                  ↓
                                                           trạng thái ra
~~~

Sơ đồ này có vẻ phức tạp hơn các chương trước.

Nhưng bạn chưa cần nhớ chi tiết.

Chỉ cần nhìn thấy ba ý:

~~~text
lấy thông tin từ vị trí khác
          +
biến đổi trạng thái
          +
giữ đường tín hiệu cũ
~~~

Đó là ba hộp lớn ta sẽ lần lượt mở.

## “Tri thức” nằm ở attention hay FFN?

Đây là câu hỏi hấp dẫn nhưng quá sớm.

Ta chưa nên viết:

> attention là nơi hiểu ngữ cảnh.

hay:

> FFN là nơi lưu tri thức.

Các thành phần tương tác với nhau qua nhiều lớp.

Trọng số đã học nằm ở nhiều nơi.

Trạng thái thay đổi theo ngữ cảnh.

Vì vậy cuốn sách sẽ tránh biến một thành phần thành “chiếc tủ chứa tri thức” chỉ vì ta dễ hình dung như vậy.

Ta sẽ hỏi câu nhỏ hơn:

> Mỗi thành phần đóng góp kiểu biến đổi nào vào đường đi của trạng thái?

Đó là câu hỏi có thể mở từng bước.

### Nhớ 3 điều

1. **Một lớp Transformer gồm nhiều bước, không phải một phép tính duy nhất.**
2. **Cơ chế chú ý (attention) giúp các vị trí trao đổi thông tin; mạng truyền thẳng (FFN) tiếp tục biến đổi trạng thái; đường cộng tắt (residual connection) giữ một đường cho tín hiệu cũ đi tiếp.**
3. **Nhiều lớp nối tiếp nhau tạo thành một chuỗi cập nhật trạng thái, chứ không có một hộp duy nhất phải “chứa toàn bộ tri thức”.**

**Tiếp theo: [Chương 6 — Cơ chế chú ý: token nhìn những token khác như thế nào?](06-co-che-chu-y.md)**
