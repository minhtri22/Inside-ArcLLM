# Lời mở đầu — Đi cùng một token

Bạn mở một ứng dụng AI và gõ:

~~~text
Hà Nội là thủ đô của nước nào?
~~~

Vài giây sau, màn hình hiện:

~~~text
Việt Nam.
~~~

Nhìn từ bên ngoài, quá trình chỉ có hai đầu:

~~~text
câu hỏi
↓
câu trả lời
~~~

Cuốn sách này chọn một cách nhìn khác.

Ta lấy một mảnh rất nhỏ trong quá trình đó — **một token** — rồi đi cùng nó.

Ta hỏi:

> **Từ lúc một mảnh văn bản bước vào mô hình cho tới khi một token mới xuất hiện, điều gì thật sự thay đổi?**

Và xa hơn:

> **Nếu ta gọi thứ đang được xử lý là “một mảnh tri thức”, cách gọi đó đúng tới đâu?**

Đó là câu hỏi của cuốn sách này.

## Trước hết, bốn khái niệm cần đủ để bắt đầu

Bạn không cần biết trước AI hoạt động như thế nào.

Ta chỉ cần bốn ý rất đơn giản.

### 1. Mô hình là phần đã học

Trong quá trình **huấn luyện (training)**, rất nhiều con số bên trong hệ thống được điều chỉnh để mô hình dự đoán tốt hơn.

Sau khi huấn luyện xong, những con số đã học được giữ lại.

Thứ chứa cấu trúc và những con số đã học đó được gọi là **mô hình (model)**.

### 2. Mô hình ngôn ngữ lớn làm việc với văn bản

Một loại mô hình chuyên xử lý và tạo ngôn ngữ được gọi là **mô hình ngôn ngữ lớn (Large Language Model, LLM)**.

Cuốn sách này nói chủ yếu về loại mô hình đó.

### 3. Transformer là kiến trúc phổ biến phía bên trong

Phần lớn mô hình ngôn ngữ lớn hiện đại dùng một họ kiến trúc gọi là **Transformer**.

Ở đây chưa cần biết Transformer hoạt động ra sao.

Chỉ cần hiểu rằng nó gồm nhiều lớp xử lý nối tiếp nhau.

Ta sẽ mở từng phần khi tới đúng chương.

### 4. Token là đơn vị đầu vào nhỏ hơn cả câu

Mô hình không xử lý cả câu như một khối nguyên vẹn.

Văn bản được chia thành những mảnh nhỏ hơn gọi là **token**.

Ngay Chương 1, ta sẽ sửa cách hiểu này cho chính xác hơn.

Bốn ý trên là đủ để bắt đầu.

Những khái niệm khác như khối số, CPU, GPU, bộ nhớ hay hệ thực thi sẽ chỉ xuất hiện khi câu chuyện thật sự cần tới chúng — và lúc đó chúng sẽ được giải thích tại chỗ.

## Một câu tưởng đơn giản

Hãy nhìn câu:

~~~text
Hà Nội là thủ đô của Việt Nam.
~~~

Ta có thể chỉ vào cụm “Hà Nội” và nói:

> Đây là một mẩu thông tin.

Từ đó rất dễ hình dung:

> Có lẽ mô hình có một chỗ nào đó lưu sẵn mẩu tri thức “Hà Nội là thủ đô của Việt Nam”, rồi khi gặp đúng token nó lấy mẩu đó ra.

Cách hình dung này rất tự nhiên.

Nhưng nó cũng rất dễ dẫn ta đi sai.

Một token không phải một chiếc hộp nhỏ chứa sẵn ý nghĩa.

Một token cũng không chạy xuyên mô hình như một kiện hàng mà bên trong giữ nguyên cùng một “mảnh tri thức”.

Trên đường đi, token được đổi thành số.

Những con số đó được biến đổi.

Chúng chịu ảnh hưởng của những token khác.

Trạng thái của chúng thay đổi qua nhiều lớp.

Cuối cùng, mô hình tạo ra một trạng thái số rồi dùng trạng thái đó để chấm điểm những token có thể xuất hiện tiếp theo.

Vậy “tri thức” nằm ở đâu?

Đây chính là nơi câu chuyện bắt đầu trở nên thú vị.

## Nhìn thử một token đi qua mô hình

Trước khi đi sâu, ta thử nhìn toàn bộ quá trình bằng một hình thật đơn giản.

Trong hệ thống thật, văn bản **không nhất thiết được tách đúng theo từng từ**.

Ví dụ câu:

~~~text
Hà Nội là thủ đô của ...
~~~

có thể được một bộ tách chia theo những cách như:

~~~text
Hà Nội | là | thủ đô | của | ...
~~~

hoặc:

~~~text
Hà | Nội | là | thủ đô | của | ...
~~~

hoặc một cách khác nữa.

Ta sẽ học chuyện đó kỹ hơn ở Chương 1.

**Nhưng riêng trong hình dưới đây, để dễ nhìn, ta tạm coi mỗi từ là một token.**

~~~text
┌──────────────────────────────────────────────────────────────────┐
│ Ví dụ trực giác: một chuỗi token đi qua mô hình như thế nào?   │
├──────────────────────────────────────────────────────────────────┤
│ Câu đang có:                                                     │
│                                                                  │
│ "Hà Nội là thủ đô của ..."                                      │
│                                                                  │
│ Tạm coi mỗi từ là một token:                                    │
│                                                                  │
│ [Hà] [Nội] [là] [thủ] [đô] [của]                               │
│   │    │     │    │     │     │                                  │
│   └────┴─────┴────┴─────┴─────┘                                  │
│                ↓                                                 │
│      đi qua nhiều lớp xử lý                                     │
│                ↓                                                 │
│                                                                  │
│ Lớp 1: trạng thái số của các vị trí bắt đầu thay đổi            │
│ Lớp 2: các trạng thái tiếp tục chịu ảnh hưởng lẫn nhau          │
│ ...                                                              │
│ Lớp N: mô hình có một trạng thái mới cho toàn chuỗi hiện tại    │
│                                                                  │
│                ↓                                                 │
│      chấm điểm token có thể đứng tiếp                            │
│                ↓                                                 │
│             [Việt]                                               │
│                                                                  │
│ Chuỗi trở thành:                                                 │
│ [Hà] [Nội] [là] [thủ] [đô] [của] [Việt]                        │
│                                                                  │
│                ↓                                                 │
│      chạy thêm một bước nữa                                     │
│                ↓                                                 │
│              [Nam]                                               │
│                                                                  │
│ Kết quả:                                                         │
│ "Hà Nội là thủ đô của Việt Nam."                                │
└──────────────────────────────────────────────────────────────────┘
~~~

Có hai điều rất quan trọng trong hình này.

**Thứ nhất:** token “Hà” không biến thành một token khác sau mỗi lớp.

Thứ thay đổi là **trạng thái số** mà mô hình đang giữ cho vị trí đó.

**Thứ hai:** mô hình thường sinh **một token tiếp theo**, nối token đó vào chuỗi, rồi lặp lại quá trình.

Vì vậy một câu trả lời dài có thể hình dung như:

~~~text
chuỗi hiện tại
↓
sinh 1 token mới
↓
nối vào chuỗi
↓
sinh tiếp 1 token
↓
nối tiếp
↓
...
↓
một chuỗi văn bản có nghĩa
~~~

Hình này chỉ là bản đồ đầu tiên.

Nó chưa nói token tương tác bằng cơ chế nào, trạng thái số gồm những gì hay vì sao “Việt” được chấm điểm cao hơn những token khác.

Đó chính là những chiếc hộp mà các chương sau sẽ lần lượt mở ra.

## Tên sách là một câu hỏi

Cuốn sách có tên:

# **TOKEN — Đường đi của mảnh tri thức**

Nhưng tên đó không có nghĩa:

~~~text
1 token = 1 mảnh tri thức
~~~

Ngược lại.

Một trong những việc đầu tiên ta sẽ làm là phá bỏ cách hiểu đó.

Tên sách muốn đặt ra câu hỏi:

> **Khi ta có cảm giác một “mảnh tri thức” đang đi xuyên qua mô hình, thứ thật sự đang thay đổi là gì?**

Có thể đó là:

- cách token được biểu diễn;
- ảnh hưởng của ngữ cảnh;
- trạng thái trung gian;
- các trọng số đã học;
- sự tương tác giữa nhiều lớp;
- hoặc chính cách con người diễn giải đầu ra.

Có lẽ “tri thức” không nằm ở một điểm duy nhất.

Có lẽ nó chỉ hiện ra từ sự tương tác của rất nhiều phần.

Ta sẽ không trả lời bằng một khẩu hiệu.

Ta sẽ đi từng bước.

## Hai con đường chạy song song

Cuốn sách có hai câu chuyện.

Câu chuyện thứ nhất nằm **bên trong mô hình**:

~~~text
văn bản
↓
token
↓
mã số
↓
biểu diễn bằng số
↓
biến đổi qua nhiều lớp
↓
trạng thái cuối
↓
token tiếp theo
~~~

Câu chuyện thứ hai nằm **ở phía người quan sát**:

~~~text
hiện tượng
↓
dấu vết
↓
phép đo
↓
bằng chứng
↓
diễn giải
↓
điều ta có quyền kết luận
~~~

Hai con đường này sẽ gặp nhau ở nửa sau cuốn sách.

Bởi khi ta nói:

> “Phần này mất nhiều thời gian hơn.”

hay:

> “Bộ đếm phần cứng thay đổi như thế này.”

ta không chỉ nói về mô hình.

Ta còn nói về **khả năng quan sát của chính mình**.

Và ở đó xuất hiện một nguyên tắc rất quan trọng:

> **Điều ta đo được không tự động bằng điều đang tồn tại trong thực tế.**

Một bộ đếm có thể trả về 0.

Một khác biệt có thể rất nhỏ.

Hai tín hiệu có thể gần như giống nhau.

Một khuôn mẫu có thể trông rất thuyết phục.

Nhưng mỗi lần như vậy, ta vẫn phải hỏi:

> Phép đo này có đủ để nói điều mình đang muốn nói không?

## Các ví dụ thực tế đến từ đâu?

Một số chương dùng những con số lấy từ thí nghiệm thật.

Nhưng cuốn sách không kể lịch sử phát triển công cụ đã tạo ra những phép đo đó.

Tên công cụ, cấu trúc phần mềm, nhánh nghiên cứu, quy trình triển khai và những cơ chế nội bộ không cần thiết cho việc học kiến thức sẽ không xuất hiện trong sách.

Ta chỉ giữ lại điều có giá trị khoa học rộng hơn:

- một phép tính logic có thể khác một công việc vật lý như thế nào;
- thời gian có thể tập trung ở đâu;
- một phép đo có thể thất bại ra sao;
- tại sao một con số bằng 0 chưa chắc đại lượng thật bằng 0;
- vì sao một khuôn mẫu mạnh vẫn chưa đủ để nói về nguyên nhân.

Nói ngắn gọn:

> **Thí nghiệm là nguồn bằng chứng, không phải nhân vật chính của cuốn sách.**

## Cách đọc thuật ngữ trong sách

Khi có cách gọi tiếng Việt đủ rõ, sách dùng dạng:

> **tiếng Việt (English)**

Ví dụ:

> **cách biểu diễn (representation)**

> **trạng thái ẩn (hidden state)**

> **chương trình GPU (kernel)**

Mục tiêu là để bạn hiểu câu bằng tiếng Việt trước, nhưng vẫn nhận ra đúng từ chuyên ngành khi gặp tài liệu khác.

Với những từ đã trở thành tên gọi phổ biến như **token**, sách sẽ giải thích thật rõ ý nghĩa rồi giữ nguyên từ đó.

## Và ta bắt đầu ở đâu?

Bắt đầu từ thứ nhỏ nhất.

Một token.

Nhưng ngay chương đầu tiên, ta sẽ phát hiện rằng thứ tưởng nhỏ và đơn giản đó đã phức tạp hơn trực giác của mình.

Bởi câu hỏi đầu tiên không phải:

> Token đi đâu?

Mà là:

> **Token thực sự là gì?**

**Tiếp theo: [Chương 1 — Token không phải là một từ](01-token-khong-phai-la-mot-tu.md)**
