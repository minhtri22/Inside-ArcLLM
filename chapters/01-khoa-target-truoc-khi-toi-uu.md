# Chương 1 — Bên dưới một câu trả lời AI có gì?

> **Mức đọc: Đi sâu**
>
> **Bạn đang mở phần nào của cỗ máy?**
>
> ```text
> Câu hỏi của người dùng
>         ↓
>      Mô hình
>         ↓
>    Hệ thực thi
>         ↓
>   CPU / GPU / bộ nhớ
> ```
>
> Chương này chỉ cần làm rõ bốn tầng trên. Những phép tính bên trong mô hình sẽ được mở ở các chương sau.

Bạn gõ:

```text
Hà Nội là thủ đô của nước nào?
```

Ứng dụng AI trả lời:

```text
Việt Nam.
```

Nếu chỉ dùng AI, ta không cần biết điều gì xảy ra ở giữa.

Nhưng nếu muốn tự xây cỗ máy, ta phải phân biệt những phần có vai trò rất khác nhau.

## Một ví dụ đời thường trước khi nói kỹ thuật

Hãy tưởng tượng một công trường.

Có:

1. **bản thiết kế**, chứa thông tin về thứ cần được tạo ra;
2. **đội tổ chức thi công**, đọc bản thiết kế, chia việc và quyết định thứ tự;
3. **máy móc và người thực hiện**, thật sự làm các công việc cụ thể.

Trong một hệ AI, phép so sánh gần đúng là:

```text
bản thiết kế đã chứa những gì học được
→ mô hình

đội tổ chức thi công
→ hệ thực thi

máy móc làm việc
→ CPU / GPU / bộ nhớ
```

Đây chỉ là ví dụ để nắm vai trò. Mô hình AI không phải một bản vẽ xây nhà, và GPU không phải máy xúc. Nhưng phép so sánh giúp ta tránh nhầm ba thứ thành một.

## Mô hình không phải ứng dụng chat

Ứng dụng mà bạn nhìn thấy có thể làm rất nhiều việc:

- hiển thị ô chat;
- lưu lịch sử;
- tải tệp;
- tìm kiếm Internet;
- gọi công cụ khác;
- quản lý tài khoản.

**Mô hình (model)** nằm ở một lớp khác.

Nó chứa cấu trúc và rất nhiều con số đã được điều chỉnh trong quá trình huấn luyện. Khi nhận đầu vào mới, những con số đó được dùng trong các phép tính để tạo ra token tiếp theo.

Ta có thể vẽ:

```text
Người dùng
    ↓
Ứng dụng AI
    ↓
Mô hình
```

Nếu tắt giao diện chat nhưng vẫn có cách đưa dữ liệu vào mô hình, mô hình vẫn có thể chạy.

Ngược lại, chỉ có giao diện đẹp mà không có mô hình phía sau thì ứng dụng không thể tự sinh câu trả lời.

## Nhưng mô hình nằm trên ổ đĩa vẫn chưa tự chạy được

Giả sử bạn tải về một tệp mô hình.

Tệp đó nằm trên ổ SSD.

Nó không thể tự nói:

> “Hãy lấy khối số này, đưa vào GPU, thực hiện phép nhân, giữ kết quả ở bộ nhớ này, rồi chuyển sang bước tiếp theo.”

Một tệp chỉ là dữ liệu.

Cần một chương trình hiểu cách đọc nó và tổ chức việc thực hiện.

Chương trình đó được gọi là **hệ thực thi (runtime)**.

Ta thêm một tầng:

```text
Người dùng
    ↓
Ứng dụng AI
    ↓
Mô hình
    ↓
Hệ thực thi
```

Hệ thực thi làm những việc như:

- mở tệp mô hình;
- tìm đúng dữ liệu cần dùng;
- chuẩn bị bộ nhớ;
- gửi phép tính xuống CPU hoặc GPU;
- giữ lại những kết quả cần cho bước tiếp theo;
- lặp lại quá trình cho tới khi sinh được token mới.

Vì vậy:

> **Mô hình là thứ đã học. Hệ thực thi là thứ làm cho mô hình chạy.**

Đây là ranh giới quan trọng nhất của Chương 1.

## CPU và GPU nằm ở đâu?

Hệ thực thi không tự làm mọi phép tính bằng ý nghĩ.

Cuối cùng công việc phải chạy trên phần cứng.

**CPU — bộ xử lý trung tâm** là bộ xử lý đa dụng của máy tính.

**GPU — bộ xử lý đồ họa** có khả năng thực hiện rất nhiều phép tính số tương tự nhau song song, nên đặc biệt hữu ích với nhiều phép tính trong mô hình ngôn ngữ.

Bộ nhớ giữ dữ liệu đang được sử dụng.

Bây giờ bức tranh đầy đủ của chương là:

```text
Người dùng
    ↓
Ứng dụng AI
    ↓
Mô hình
    ↓
Hệ thực thi
    ↓
CPU / GPU / bộ nhớ
```

ArcLLM nằm chủ yếu ở tầng **hệ thực thi**.

## Một token là gì?

Ta sẽ dùng từ này rất nhiều nên cần hiểu đủ sớm.

Hãy bắt đầu bằng cách hiểu đơn giản:

> **Token là một mảnh văn bản nhỏ mà mô hình xử lý.**

Nếu có câu:

```text
Tôi thích cà phê.
```

người mới có thể tạm hình dung:

```text
Tôi | thích | cà | phê | .
```

Nhưng đây chỉ là hình dung.

Trong hệ thống thật, token **không nhất thiết là một từ**.

Một token có thể là cả từ, một phần của từ, dấu câu hoặc một chuỗi ký tự khác.

Một **bộ tách và mã hóa văn bản (tokenizer)** quyết định cách chia và đổi mỗi token thành một mã số.

Ví dụ minh họa:

```text
"ChatGPT!"
    ↓
bộ tách và mã hóa
    ↓
Chat | GPT | !
    ↓
314 | 9821 | 17
```

Các số trên chỉ là ví dụ.

Điều cần nhớ:

> **Mô hình không trực tiếp cầm câu chữ như con người. Nó nhận những mã số đại diện cho các token.**

## ArcLLM là gì?

ArcLLM là dự án nghiên cứu lớp hệ thực thi vừa nói.

Dự án bắt đầu trên một máy tính dùng bộ xử lý Intel Core Ultra 7 258V và GPU tích hợp Intel Arc 140V.

Tên dự án ghép từ:

```text
Intel Arc
+
LLM
↓
ArcLLM
```

**LLM** là viết tắt của *Large Language Model*, tức **mô hình ngôn ngữ lớn**.

Tên ArcLLM ghi lại chiếc máy nơi dự án bắt đầu. Nó không có nghĩa hệ thực thi này về nguyên tắc chỉ được phép chạy trên Intel Arc.

## Tại sao lại tự xây khi đã có llama.cpp và Ollama?

Nếu mục tiêu chỉ là:

> “Tôi muốn tải một mô hình về và trò chuyện ngay.”

thì tự xây ArcLLM là cách vòng vèo.

llama.cpp đã là một hệ thực thi trưởng thành. Ollama giúp người dùng tải, quản lý và chạy mô hình thuận tiện hơn.

ArcLLM không ra đời vì những phần mềm đó “không tốt”.

Nó ra đời vì câu hỏi khác:

> **Nếu tự xây từ những phần thấp nhất, ta có hiểu được vì sao một hệ thực thi phải có hình dạng như hiện nay không?**

Một hệ thống trưởng thành đã chứa rất nhiều quyết định tích lũy qua nhiều năm.

Nếu dùng nó làm lõi ngay từ đầu, ta có thể đo và sửa, nhưng nhiều câu hỏi đã được trả lời hộ.

ArcLLM chọn con đường chậm hơn:

```text
cần đọc tệp mô hình
→ tự xây phần đọc

cần dùng GPU
→ tự xây đường giao việc cho GPU

cần một phép tính
→ tự xây phép tính đó

một cách làm thất bại
→ giữ lại thất bại

bằng chứng buộc kiến trúc đổi
→ mới đổi kiến trúc
```

Tinh thần là:

> **Muốn hiểu chiếc máy, hãy thử tự xây chiếc máy.**

## llama.cpp vẫn rất quan trọng

Tự xây không có nghĩa tự so với chính mình mãi mãi.

Hãy tưởng tượng ta tự chế một chiếc xe.

Hôm qua nó chạy 10 km/h.

Hôm nay sửa xong chạy 20 km/h.

Ta có thể nói:

> “Nhanh gấp đôi hôm qua.”

Nhưng nếu một chiếc xe bình thường chạy 100 km/h, ta vẫn còn cách rất xa thực tế.

ArcLLM cũng vậy.

Một cải tiến 2 lần hay 3 lần so với phiên bản trước có thể cho biết thay đổi vừa làm có tác dụng.

Nhưng nó **không tự động chứng minh ArcLLM nhanh hơn một hệ thực thi trưởng thành**.

Vì vậy llama.cpp được dùng như **mốc đối chứng bên ngoài**.

Muốn so công bằng, hai bên phải dùng cùng mô hình, cùng máy, cùng đầu vào và cùng cách đo.

Chuyện này sẽ trở thành trọng tâm của Chương 9.

## Nhưng trước khi chạy nhanh, phải chắc rằng ta đang chạy đúng thứ

Đây là bước đầu tiên thật sự của dự án.

Giả sử hôm nay ta dùng tệp mô hình A.

Ngày mai vô tình đổi sang tệp B.

Kết quả nhanh hơn 10%.

Ta không còn biết:

> Hệ thực thi tốt hơn?

hay:

> Tệp mô hình đã khác?

Vì vậy ArcLLM bắt đầu bằng một nguyên tắc rất đơn giản:

> **Khóa đúng đối tượng trước khi tối ưu.**

Dự án chọn một mô hình cụ thể: Qwen2.5-Coder-1.5B.

Nhưng tên tệp vẫn chưa đủ. Hai tệp có thể cùng tên mà nội dung khác nhau.

Vì vậy tệp được kiểm tra bằng **SHA-256**, một cách tạo “dấu vân tay số” từ nội dung dữ liệu.

Nếu nội dung đổi, dấu vân tay gần như chắc chắn cũng đổi.

Nhờ đó ta biết các lần thử đang dùng đúng cùng một tệp.

## Máy có đủ khả năng để bắt đầu không?

Khóa đúng tệp vẫn chưa đủ.

Nếu mô hình cần nhiều bộ nhớ hơn chiếc máy có thể cung cấp, mọi kế hoạch phía sau đều vô nghĩa.

Kiểm tra đầu tiên của ArcLLM cho thấy tệp mô hình có **338 khối số (tensor)**.

Ở đây hãy tạm hiểu khối số là những dãy hoặc bảng số mà mô hình dùng trong phép tính. Chương 2 sẽ mở chúng ra kỹ hơn.

Với phạm vi làm việc 4.096 token, bộ lập kế hoạch ước lượng cần khoảng:

```text
2,120 GiB bộ nhớ
```

trong khi mức dung lượng có thể đáp ứng đã được xác nhận là:

```text
3,75 GiB
```

**GiB (gibibyte)** là một đơn vị dung lượng. Ở đây ta chưa cần học cách đổi đơn vị; chỉ cần so:

```text
2,120 < 3,75
```

Về dung lượng thiết kế, nó vừa.

## “Máy đang bận” khác “thiết kế không vừa”

Có một lần chạy bị chặn vì lúc đó lượng RAM còn trống thấp hơn mức dự phòng 8 GiB.

Thoạt nhìn, có thể kết luận:

> “Máy không đủ bộ nhớ.”

Nhưng đó chưa phải kết luận đúng.

Có hai chuyện khác nhau:

```text
Máy đang thiếu bộ nhớ lúc này
```

và:

```text
Thiết kế luôn cần nhiều bộ nhớ hơn máy có thể cung cấp
```

Trường hợp đầu có thể chỉ vì đang mở nhiều chương trình.

Trường hợp sau mới là giới hạn của thiết kế.

ArcLLM phải phân biệt hai chuyện đó trước khi quyết định có dừng hướng nghiên cứu hay không.

Sau khi ranh giới này được làm rõ, bước đầu tiên được đánh giá **ĐẠT (PASS)**.

Chưa có một mô hình hoàn chỉnh đang trò chuyện.

Chưa có phép tính GPU tối ưu.

Chưa có những lớp xử lý mà ta sẽ gặp sau này.

Nhưng ta đã biết chắc hơn ba điều:

```text
đang dùng đúng mô hình nào
↓
đúng tệp nào
↓
trên chiếc máy nào
```

Mọi phép đo về sau phụ thuộc vào nền móng này.

### Nhớ 3 điều

1. **Mô hình là phần chứa những gì đã học; hệ thực thi là phần làm cho mô hình chạy; CPU/GPU là phần cứng thực hiện phép tính.**
2. **ArcLLM tự xây hệ thực thi để hiểu từng quyết định bên trong, không phải vì các công cụ như llama.cpp hay Ollama không tốt.**
3. **Trước khi tối ưu, phải khóa đúng mô hình, đúng tệp và đúng điều kiện phần cứng. Nếu không, một con số đẹp hơn chưa chắc nói lên điều gì.**

**Tiếp theo: [Chương 2 — Bên trong tệp mô hình có gì?](02-gguf-tensor-store.md)**

Ta vừa biết tệp nghiên cứu chứa 338 khối số.

Chương sau sẽ mở tệp đó ra và trả lời: một “khối số” thực sự được lưu như thế nào, vì sao có loại chiếm nhiều chỗ hơn loại khác, và vì sao cách lưu các con số quyết định trực tiếp liệu mô hình có vừa trong bộ nhớ hay không.
