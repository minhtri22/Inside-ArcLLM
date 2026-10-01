# Phần 0 — Bản đồ trânh lạc lối

> **Mức đọc: Nền tảng**
>
> Nếu bạn chưa biết AI, mô hình, token, RAM, CPU hay GPU là gì, hãy bắt đầu ở đây. Không cần ghi nhớ thuật ngữ. Mục tiêu chỉ là hiểu từng vai trò một.

## Bắt đầu từ thứ quen thuộc nhất

Bạn mở một ứng dụng AI và gõ:

```text
Hà Nội là thủ đô của nước nào?
```

Vài giây sau, màn hình hiện:

```text
Việt Nam.
```

Từ góc nhìn của người dùng, quá trình chỉ có hai đầu:

```text
câu hỏi
↓
câu trả lời
```

Cuốn sách này muốn mở phần ở giữa.

Nhưng ta sẽ không mở tất cả cùng lúc.

Trước tiên, hãy chia hệ thống thành ba vai trò rất đơn giản:

```text
1. Thứ chứa những gì máy đã học
2. Thứ biết cách làm cho nó chạy
3. Máy móc thật sự thực hiện phép tính
```

Sau khi hiểu ba vai trò đó, ta mới đặt tên.

## 1. Mô hình: thứ chứa những gì đã học

Hãy tưởng tượng một chiếc máy có hàng tỷ núm nhỏ.

Trong quá trình học, các núm đó được điều chỉnh rất nhiều lần.

Sau khi học xong, vị trí của những núm ấy được giữ lại.

Khi có câu hỏi mới, máy không học lại từ đầu. Nó dùng những giá trị đã được điều chỉnh để tính phần tiếp theo.

Trong AI, thứ chứa cấu trúc và những con số đã học đó được gọi là **mô hình (model)**.

Đây chỉ là hình dung ban đầu, nhưng đủ để đi tiếp.

Một mô hình không phải là cả ứng dụng chat.

Ứng dụng còn có thể có giao diện, lịch sử trò chuyện, tìm kiếm, công cụ và nhiều phần khác.

Ta có thể tạm vẽ:

```text
Ứng dụng AI
    ↓
Mô hình
```

### Những con số đã học được gọi là gì?

Trong quá trình **huấn luyện (training)**, rất nhiều con số bên trong mô hình được điều chỉnh.

Những con số có thể học được gọi chung là **tham số (parameter)**.

Một loại tham số rất phổ biến là **trọng số (weight)**.

Ở mức nhập môn, bạn chỉ cần nhớ:

```text
huấn luyện
↓
điều chỉnh rất nhiều con số
↓
các tham số / trọng số
↓
được lưu lại trong mô hình
```

Khi dùng mô hình đã học để xử lý một câu hỏi mới, ta gọi quá trình đó là **suy luận (inference)**.

ArcLLM chủ yếu nghiên cứu giai đoạn suy luận.

## 2. Token: mảnh văn bản mà mô hình xử lý

Bây giờ ta có một câu:

```text
Hà Nội rất đẹp.
```

Nếu mới học, bạn có thể **tạm hình dung** mỗi từ là một mảnh:

```text
Hà | Nội | rất | đẹp | .
```

Mô hình không nhất thiết chia đúng như vậy, nhưng cách hình dung này giúp ta hiểu ý tưởng đầu tiên.

Mỗi mảnh văn bản mà mô hình xử lý được gọi là một **token**.

Bây giờ ta sửa cách hiểu cho chính xác hơn một chút:

> **Token không nhất thiết là một từ.**

Tùy mô hình, một token có thể là:

- cả một từ;
- một phần của từ;
- một dấu câu;
- một chuỗi ký tự;
- hoặc một phần có liên quan tới khoảng trắng.

Phần mềm chia văn bản thành các token được gọi là **bộ tách và mã hóa văn bản (tokenizer)**.

Ví dụ minh họa:

```text
"ChatGPT!"
     ↓
bộ tách và mã hóa
     ↓
"Chat" | "GPT" | "!"
```

Một mô hình khác có thể chia khác.

Điều cần nhớ lúc này chỉ là:

> **Con người nhìn thấy câu chữ; mô hình xử lý một chuỗi token.**

## 3. Mỗi token được đổi thành số

Máy tính không làm phép tính trực tiếp trên chữ “Hà” hay “Nội”.

Mỗi token được gán một mã số.

Ta gọi nó là **mã token (token ID)**.

Ví dụ tưởng tượng:

```text
"Hà"  → 314
"Nội" → 982
```

Các con số này chỉ để minh họa.

Sau đó mã token lại được đổi thành một dãy số phù hợp với các phép tính của mô hình.

Bước đổi đó gọi là **nhúng (embedding)**.

Có thể hình dung:

```text
token
↓
mã token
↓
một dãy số
```

Những dãy và bảng số như vậy thường được gọi là **khối số (tensor)**.

Từ “tensor” nghe khó, nhưng ở đây chưa cần toán học.

Hãy tạm hiểu:

> **Khối số là một nhóm các con số được sắp theo một hình dạng để máy tính thực hiện phép tính.**

Một hàng số là trường hợp đơn giản.

Một bảng số là một trường hợp khác.

Sau này ta mới cần những hình dạng phức tạp hơn.

## 4. Mô hình xử lý những khối số bằng nhiều lớp

Các khối số không đi thẳng tới câu trả lời.

Chúng đi qua nhiều **lớp xử lý (layer)**.

Bạn có thể hình dung giống một dây chuyền:

```text
dữ liệu đầu vào
↓
lớp 1
↓
lớp 2
↓
lớp 3
↓
...
↓
dữ liệu đầu ra
```

Phần lớn mô hình ngôn ngữ lớn hiện đại thuộc một họ kiến trúc gọi là **Transformer**.

Ta chưa cần học công thức Transformer.

Chỉ cần biết mô hình ArcLLM nghiên cứu dùng nhiều **lớp giải mã (decoder layer)** nối tiếp nhau để dần biến đổi trạng thái của token.

Bên trong mỗi lớp có những nhóm phép tính mà sau này ta sẽ gặp như:

- chuẩn hóa tín hiệu;
- cơ chế chú ý (attention);
- nhánh biến đổi tín hiệu (feed-forward network, FFN);
- đường cộng lại tín hiệu cũ.

Chúng ta chưa cần hiểu chúng ở đây.

## 5. Mô hình “đặc” nghĩa là gì?

Một từ bạn có thể gặp khi đọc về AI là **mô hình đặc (dense model)**.

Ở mức đơn giản:

> Trong một mô hình đặc truyền thống, mỗi token đi qua cùng chuỗi lớp xử lý chính.

Có thể hình dung:

```text
token
↓
lớp 1
↓
lớp 2
↓
lớp 3
↓
...
```

Có những kiến trúc khác, chẳng hạn mô hình có nhiều “chuyên gia” và chỉ chọn một số nhánh cho mỗi token. Nhưng đó chưa phải điều ta cần tập trung trong cuốn sách này.

Điều quan trọng là đừng nhầm:

- **đặc hay nhiều chuyên gia** nói về cách tổ chức tính toán;
- **Q4, Q6, F32** mà ta gặp sau này nói về cách biểu diễn các con số.

## 6. CPU và GPU: nơi phép tính thật sự xảy ra

Tới đây ta có mô hình và rất nhiều con số.

Nhưng một tệp mô hình nằm trên ổ đĩa không tự tính được.

Máy tính cần bộ xử lý.

**bộ xử lý trung tâm (Central Processing Unit, CPU)** là bộ xử lý đa dụng. Nó giỏi làm nhiều loại công việc và điều phối hệ thống.

**bộ xử lý đồ họa (Graphics Processing Unit, GPU)** ban đầu nổi tiếng nhờ xử lý hình ảnh, nhưng nó cũng rất phù hợp với việc thực hiện rất nhiều phép tính số tương tự nhau song song.

Mô hình ngôn ngữ cần vô số phép nhân và cộng trên các khối số lớn. Vì thế GPU thường rất quan trọng.

Ta còn cần **bộ nhớ** để giữ dữ liệu đang được sử dụng.

Một bức tranh đơn giản:

```text
Ổ lưu trữ
giữ tệp mô hình lâu dài
        ↓
Bộ nhớ
giữ dữ liệu đang cần dùng (RAM)
        ↓
CPU / GPU
thực hiện phép tính
```

## 7. Vậy ai tổ chức tất cả những việc đó?

Đây là chỗ dễ nhầm nhất.

Tệp mô hình không tự biết:

- phải đọc khối số nào trước;
- đặt dữ liệu ở đâu trong bộ nhớ;
- phép tính nào gửi cho CPU;
- phép tính nào gửi cho GPU;
- kết quả của bước này phải chuyển sang bước nào tiếp theo.

Cần một chương trình đứng giữa mô hình và phần cứng để tổ chức những việc đó.

Chương trình như vậy được gọi là **hệ thực thi (runtime)**.

Có thể dùng một phép so sánh đơn giản:

```text
Mô hình
≈ bản thiết kế + những con số đã học

Hệ thực thi
≈ đội biết đọc bản thiết kế và tổ chức công việc

CPU / GPU
≈ máy móc thật sự thực hiện các phép tính
```

Phép so sánh này không hoàn hảo, nhưng đủ để phân biệt ba vai trò.

Đây cũng là lý do một tệp mô hình nằm yên trên ổ đĩa chưa phải một hệ AI đang hoạt động.

## 8. LLM nằm ở đâu trong thế giới AI?

Bây giờ mới cần một bản đồ tên gọi.

**Trí tuệ nhân tạo (Artificial Intelligence, AI)** là chiếc ô rất rộng.

Bên trong đó có **học máy (Machine Learning)**: thay vì chỉ viết sẵn mọi luật, ta để hệ thống học mẫu từ dữ liệu.

Một họ quan trọng của học máy là **mạng nơ-ron nhân tạo (Neural Network)**.

Một mô hình chuyên xử lý ngôn ngữ được gọi là **mô hình ngôn ngữ (Language Model)**.

Khi quy mô của mô hình ngôn ngữ rất lớn, ta thường gọi nó là **mô hình ngôn ngữ lớn (Large Language Model, LLM)**.

Có thể vẽ:

```text
Trí tuệ nhân tạo
↓
Học máy
↓
Mạng nơ-ron
↓
Mô hình ngôn ngữ
↓
Mô hình ngôn ngữ lớn (LLM)
```

Phần lớn LLM hiện đại dùng kiến trúc Transformer.

Mô hình trong hành trình ArcLLM thuộc loại Transformer chỉ dùng phần giải mã để sinh token tiếp theo.

Tên kỹ thuật đầy đủ là **Transformer chỉ có bộ giải mã (decoder-only Transformer)**.

Bạn không cần nhớ tên đó ngay.

## 9. Một câu đi qua cỗ máy như thế nào?

Bây giờ các từ trong sơ đồ sau đều đã xuất hiện ít nhất một lần:

```text
Văn bản
   ↓
Bộ tách và mã hóa
   ↓
Token
   ↓
Mã token
   ↓
Khối số
   ↓
Nhiều lớp xử lý của mô hình
   ↓
Điểm số cho các token có thể đứng tiếp
   ↓
Chọn token tiếp theo
   ↓
Lặp lại
   ↓
Câu trả lời
```

Trong toàn bộ quá trình đó:

```text
Hệ thực thi
↓
tổ chức bộ nhớ và thứ tự công việc
↓
CPU / GPU
↓
thực hiện các phép tính thật
```

Đây là bản đồ nền của cuốn sách.

Các chương sau không bắt bạn học thêm toàn bộ một bản đồ mới. Chúng chỉ mở từng hộp khi tới lượt.

## 10. Còn lượng tử hóa là gì?

Một mô hình có hàng tỷ tham số.

Nếu mỗi con số đều được lưu bằng nhiều bit, tệp sẽ rất lớn và việc chuyển dữ liệu cũng tốn kém.

**Lượng tử hóa (quantization)** là cách biểu diễn một số trọng số bằng ít bit hơn để tiết kiệm dung lượng và giảm lượng dữ liệu phải xử lý, đổi lại chấp nhận một mức xấp xỉ có kiểm soát.

Bạn sẽ gặp những tên như F32, Q4_K và Q6_K.

Chưa cần học chúng ở đây.

Chương 2 sẽ bắt đầu từ bit, byte và kích thước tệp rồi giải thích từng bước.

## Trước khi sang Chương 1, chỉ cần nhớ 7 điều

1. **Mô hình** là phần chứa cấu trúc và những con số đã học.
2. **Token** là mảnh văn bản mà mô hình xử lý; nó không nhất thiết là một từ.
3. Token được đổi thành số, rồi thành các **khối số** để tính toán.
4. Dữ liệu đi qua nhiều **lớp xử lý** của mô hình.
5. **CPU/GPU** là phần cứng thật sự làm phép tính.
6. **Hệ thực thi** là chương trình tổ chức việc đọc mô hình, bộ nhớ và các phép tính.
7. ArcLLM nghiên cứu chính lớp hệ thực thi đó.

Nếu bạn hiểu được bảy ý trên dù chưa nhớ tên tiếng Anh, bạn đã đủ nền để bước vào Chương 1.

**Tiếp theo: [Chương 1 — Bên dưới một câu trả lời AI có gì?](01-khoa-target-truoc-khi-toi-uu.md)**
