# Bonus — Từ xây cỗ máy tới lắng nghe cỗ máy

> **Mức đọc: Nâng cao**
>
> Phần Bonus không mở thêm một nhánh kỹ thuật của ArcLLM. Nó chỉ giữ lại một câu hỏi xuất hiện tự nhiên sau khi ta đã nhìn đủ sâu vào bên trong cỗ máy.

## Từ nhìn đầu ra tới nhìn quá trình

Trong phần lớn thời gian sử dụng AI, ta nhìn hai thứ:

```text
đầu vào
↓
đầu ra
```

Ta hỏi:

> Câu trả lời có đúng không?

Hoặc:

> Mất bao lâu để có câu trả lời?

Đó là những câu hỏi quan trọng.

Nhưng sau hành trình ArcLLM, ta đã biết phía giữa hai đầu đó không hề trống.

Một token đi qua nhiều lớp xử lý.

Nhiều khối số được đọc.

Nhiều phép tính được gửi xuống phần cứng.

Bộ nhớ thay đổi.

Những kết quả trung gian xuất hiện rồi biến mất.

Có những phần dữ liệu được giữ lại để dùng tiếp.

Vì vậy một câu hỏi mới xuất hiện:

> **Nếu không chỉ nhìn đầu vào và đầu ra, ta có thể quan sát chính quá trình đang diễn ra bên trong hay không?**

## Quan sát trước, giải thích sau

Đây là nơi rất dễ đi quá nhanh.

Giả sử ta đo được:

```text
vùng A mất nhiều thời gian hơn vùng B
```

Điều đó cho phép nói:

> A đang tốn nhiều thời gian hơn B trong phép đo này.

Nhưng chưa cho phép nói:

> A chậm vì nguyên nhân X.

Hai câu khác nhau.

Tương tự, nếu một chỉ số phần cứng thay đổi cùng lúc với độ trễ, ta mới có một mối liên hệ quan sát được.

Muốn nói về nguyên nhân, ta cần thêm bằng chứng.

Nguyên tắc này có thể viết rất ngắn:

```text
quan sát
≠
nguyên nhân
```

Trong suốt cuốn sách, ta đã nhiều lần gặp cùng một bài học dưới những hình thức khác nhau:

- một phép tính nhanh hơn chưa có nghĩa cả hệ thống nhanh hơn;
- một con số bằng 0 chưa chắc đại lượng vật lý thật sự bằng 0;
- một thay đổi xuất hiện cùng một hiện tượng chưa chắc đã gây ra hiện tượng đó;
- một kết quả đẹp chưa chắc đến từ đúng thứ ta nghĩ mình đang đo.

Càng nhìn sâu, kỷ luật về bằng chứng càng quan trọng.

## Công cụ đo cũng có giới hạn

Có một trực giác rất dễ mắc:

> “Máy đã đưa ra con số thì con số đó phải là sự thật vật lý.”

Không hẳn.

Một phép đo luôn đi qua một **kênh đo**.

Kênh đó có thể có:

- độ phân giải hữu hạn;
- giới hạn của phần cứng;
- giới hạn của trình điều khiển;
- ảnh hưởng của cách lấy mẫu;
- chi phí do chính việc đo tạo ra.

Vì vậy nếu một bộ đếm trả về 0, có ít nhất hai khả năng rất khác:

```text
đại lượng thật sự gần bằng 0

hoặc

kênh đo không quan sát được đại lượng đó một cách đáng tin
```

Nếu không phân biệt hai khả năng này, ta có thể biến lỗi của phép đo thành một câu chuyện về hệ thống.

## Những khác biệt rất nhỏ cũng đặt ra câu hỏi

Có một lỗi ngược lại.

Khi hai giá trị gần như giống nhau, ta thường làm tròn và nói:

> “Coi như bằng nhau.”

Trong rất nhiều trường hợp, đó là lựa chọn hoàn toàn hợp lý.

Nhưng đôi khi phần chênh lệch nhỏ lại đáng để hỏi thêm.

Ví dụ:

```text
A = 1,000000
B = 1,000003
```

Ta có thể nói:

```text
A ≈ B
```

Nhưng cũng có thể hỏi:

```text
B - A = 0,000003
```

Ba phần triệu đó là gì?

Có thể chỉ là:

- sai số số học;
- nhiễu;
- giới hạn của phép đo;
- một thay đổi ngẫu nhiên.

Nhưng trước khi bỏ qua, ta có thể hỏi thêm:

> Nó có lặp lại không?

> Nó có xuất hiện ở cùng vị trí không?

> Nó có thay đổi theo thời gian theo một cách có cấu trúc không?

> Khi điều kiện đầu vào đổi nhẹ, phần chênh lệch có đổi theo một quy luật nào không?

Câu hỏi đúng không phải:

> **“Khác biệt này có đủ lớn để gây ấn tượng không?”**

Mà là:

> **“Khác biệt này có cấu trúc hay chỉ là nhiễu?”**

Ngay cả khi thấy cấu trúc, ta vẫn chưa được phép gọi đó là nguyên nhân.

Nhưng ta đã có một câu hỏi tốt hơn.

## Từ một bức ảnh tới một đoạn phim

Một phép đo tại một thời điểm giống một bức ảnh.

Nó cho ta biết trạng thái tại khoảnh khắc đó.

Nhưng hệ thống thật hoạt động theo thời gian.

Một cách nhìn khác là:

```text
trạng thái lúc 1
↓
trạng thái lúc 2
↓
trạng thái lúc 3
↓
...
```

Lúc đó ta không chỉ hỏi:

> Giá trị hiện tại là bao nhiêu?

Ta còn có thể hỏi:

> Nó đang thay đổi theo hướng nào?

> Thay đổi ở một nơi có đi kèm thay đổi ở nơi khác không?

> Có phần nào phản ứng sớm hơn phần khác không?

> Một thay đổi nhỏ có biến mất, giữ nguyên hay lan rộng?

Đây là sự chuyển từ:

```text
nhìn trạng thái
```

sang:

```text
nhìn sự thay đổi của trạng thái
```

Và đây là một cánh cửa rất rộng.

## Nhưng đừng gọi mọi dao động là tín hiệu

Khi bắt đầu nhìn vào những thay đổi nhỏ, một nguy cơ mới xuất hiện:

> **thấy pattern ở nơi thực ra chỉ có nhiễu.**

Con người rất giỏi nhìn ra hình dạng.

AI cũng rất giỏi tạo ra lời giải thích hợp lý cho một hình dạng.

Vì vậy càng đi vào những tín hiệu nhỏ, ta càng cần những câu hỏi khó hơn:

```text
có lặp lại không?
↓
có giữ được khi đổi lần chạy không?
↓
có vượt mức nhiễu của phép đo không?
↓
có xuất hiện khi điều kiện liên quan thay đổi không?
↓
có biến mất khi điều kiện đó bị loại bỏ không?
```

Một hình đẹp chưa phải bằng chứng.

Một dao động đẹp cũng chưa phải bằng chứng.

## Muốn nói “vì sao”, phải thay đổi điều gì đó

Giả sử ta nghi ngờ:

> Một thay đổi ở A gây ra thay đổi ở B.

Nếu chỉ nhìn A và B cùng biến động, ta mới có quan sát.

Một cách mạnh hơn là thay đổi A có kiểm soát, giữ những thứ khác ổn định hết mức có thể, rồi xem B có phản ứng như dự đoán hay không.

Đó là tinh thần của **can thiệp có kiểm soát**.

Có thể hình dung:

```text
trạng thái ban đầu
        ↓
thay đổi một yếu tố
        ↓
quan sát phản ứng
        ↓
so với trường hợp không thay đổi
```

Nhưng đời thật không phải lúc nào cũng cho phép can thiệp.

Không thể tùy tiện làm một hệ thống quan trọng hỏng đi chỉ để xem nó phản ứng thế nào.

Không thể tùy tiện tác động vào con người để lấy dữ liệu.

Vì vậy nhiều câu hỏi phải bắt đầu từ:

- mô phỏng;
- dữ liệu quan sát;
- môi trường thử nghiệm an toàn;
- hoặc những hệ thống số mà ta kiểm soát được.

## Một cỗ máy AI cũng là một hệ thống động

Trong ArcLLM, ta thường nói về:

```text
token
khối số
bộ nhớ
phép tính
GPU
độ trễ
```

Nhưng khi mọi thứ chạy, chúng không đứng yên.

Trạng thái thay đổi theo từng bước.

Dữ liệu được đọc rồi ghi.

Bộ nhớ đệm lớn dần.

Tín hiệu đi qua các lớp.

Phần cứng nhận những đợt công việc khác nhau.

Nhìn theo góc này, một hệ AI không chỉ là một sơ đồ các hộp nối với nhau.

Nó còn là:

> **một hệ thống có trạng thái đang thay đổi theo thời gian.**

Chỉ riêng cách nhìn đó đã mở ra nhiều câu hỏi hơn những gì cuốn sách này có thể trả lời.

## Nếu đi theo một token thì sao?

Cho tới đây, ta thường nhìn cỗ máy từ bên ngoài:

```text
phần nào tồn tại?
↓
phần nào tốn thời gian?
↓
phần nào cần thay đổi?
```

Nhưng ta cũng có thể đổi góc nhìn.

Thay vì đứng ngoài nhìn toàn bộ cỗ máy, hãy chọn một token.

Đi cùng nó.

Hỏi:

```text
nó bắt đầu từ đâu?
↓
được đổi thành những con số nào?
↓
đi qua các lớp nào?
↓
trạng thái của nó thay đổi ra sao?
↓
những phép tính logic đó biến thành
công việc vật lý như thế nào?
↓
ta đo được gì trên đường đi?
↓
và điều gì vẫn chỉ là suy đoán?
```

Đó là một câu hỏi khác với câu hỏi đã dẫn dắt *Inside ArcLLM*.

Cuốn sách này hỏi:

> **Cỗ máy được xây như thế nào?**

Câu hỏi mới là:

> **Một mảnh thông tin đi qua cỗ máy đó như thế nào?**

Không cần trả lời ngay.

Chỉ cần giữ nó lại.

## Và xa hơn nữa?

Nếu một ngày ta không chỉ đi theo một token mà muốn nhìn rất nhiều trạng thái cùng thay đổi theo thời gian, câu hỏi sẽ lại đổi.

Ta có thể bắt đầu hỏi:

> Hệ thống phản ứng như thế nào trước một thay đổi nhỏ?

> Phần thay đổi xuất hiện ở đâu trước?

> Nó tắt dần hay lan rộng?

> Những khác biệt rất nhỏ có phải chỉ là nhiễu, hay có cấu trúc lặp lại?

Đây không phải kết luận của cuốn sách.

Đây chỉ là những câu hỏi mà việc mở cỗ máy khiến ta có quyền đặt ra.

Và có lẽ đó là giá trị lớn nhất của việc hiểu một hệ thống từ bên trong:

> **Khi biết mình đang nhìn cái gì, ta bắt đầu biết nên hỏi gì tiếp theo.**

### Nhớ 3 điều

1. **Quan sát không đồng nghĩa với nguyên nhân.** Biết một hiện tượng xảy ra ở đâu hoặc đi cùng điều gì chưa đủ để nói vì sao nó xảy ra.
2. **Phép đo cũng có giới hạn.** Số 0, một khác biệt nhỏ hay một pattern đẹp đều phải được đặt trong ngữ cảnh của kênh đo và mức nhiễu.
3. **Một cỗ máy đang chạy là một hệ thống thay đổi theo thời gian.** Sau khi hiểu cấu trúc, câu hỏi tự nhiên tiếp theo là theo dấu những thay đổi đó thay vì chỉ nhìn đầu vào và đầu ra.

---

*Inside ArcLLM* kết thúc bằng việc xây và mở một cỗ máy.

Cánh cửa tiếp theo có thể bắt đầu chỉ bằng một câu hỏi:

> **Nếu chọn một token và đi cùng nó từ lúc xuất hiện cho tới khi nó để lại dấu vết trên phần cứng, ta sẽ nhìn thấy gì?**
