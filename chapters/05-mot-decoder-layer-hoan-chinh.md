# Chương 5 — Ghép các phép tính thành một lớp giải mã

> **Mức đọc: Đi sâu**
>
> **Bạn đang mở phần nào của cỗ máy?**
>
> ```text
> Các phép tính nhỏ
>         ↓
> [ một lớp giải mã ]
>         ↓
> kiểm tra đầu ra
> ```


> **Câu hỏi của chương:** Nếu từng phép tính đã đúng khi đứng riêng, khi nối chúng thành một lớp thật của mô hình thì cả chuỗi có còn đúng không?

Ở Chương 4, ArcLLM đã thử từng viên gạch.

RMSNorm được kiểm tra riêng.

Phép nhân với trọng số Q4_K được kiểm tra riêng.

RoPE, softmax, cơ chế chú ý, SwiGLU và residual cũng lần lượt được đưa xuống GPU rồi so với cách tính tham chiếu trên CPU.

Từng phép đều ĐẠT (PASS) trong phạm vi đã khóa.

Nhưng đó chưa phải một lớp giải mã.

Có một khác biệt quan trọng giữa:

```text
A đúng
B đúng
C đúng
```

và:

```text
A → B → C
cả chuỗi vẫn đúng
```

Một chiếc đồng hồ có thể gồm hàng trăm bánh răng tốt. Nhưng nếu lắp một bánh ngược chiều, chiếc đồng hồ vẫn không chạy.

P4 là lúc ArcLLM bắt đầu **lắp các bánh răng lại với nhau**.

## Lớp giải mã là gì?

Ở mức đơn giản nhất, một mô hình ngôn ngữ không xử lý văn bản bằng một phép tính duy nhất.

Dữ liệu đi qua nhiều **lớp — lớp xử lý** liên tiếp.

Mỗi lớp nhận tín hiệu từ lớp trước, thực hiện một chuỗi phép biến đổi rồi chuyển kết quả sang lớp tiếp theo.

Có thể hình dung:

```text
dữ liệu đi vào
     ↓
Lớp 0
     ↓
Lớp 1
     ↓
Lớp 2
     ↓
...
```

Trong loại mô hình mà ArcLLM đang xây hệ thực thi, mỗi lớp có hai khu vực lớn mà ta đã làm quen ở Chương 4.

Một phía là **cơ chế chú ý — phần giúp mô hình kết hợp thông tin giữa các vị trí token**.

Phía còn lại là **FFN — Feed-Forward Network, nhánh biến đổi tín hiệu sau cơ chế chú ý**.

Giữa các phần ấy còn có chuẩn hóa và những đường residual — **đường cộng tắt đưa tín hiệu cũ cộng trở lại kết quả mới**.

Nếu bỏ bớt chi tiết toán học, ta có thể nhìn một lớp như thế này:

```text
tín hiệu đi vào
      ↓
chuẩn hóa
      ↓
attention
      ↓
cộng residual
      ↓
chuẩn hóa
      ↓
FFN
      ↓
cộng residual
      ↓
tín hiệu đi ra
```

P3 đã thử các bộ phận.

P4 hỏi:

> **Nếu cho dữ liệu đi hết con đường này bằng trọng số thật của mô hình, GPU có tạo ra kết quả cuối lớp đủ gần với CPU hay không?**

## Lần này không còn dùng những mảnh rời

P4 dùng **`blk.0` — lớp đầu tiên thật của mô hình Qwen2 đã được khóa từ P0**.

Điều này rất quan trọng.

Ta không dựng một lớp đồ chơi có kích thước nhỏ rồi suy luận rằng lớp thật chắc cũng đúng.

P4 lấy các trọng số thật của lớp đó từ chính GGUF mà P1 đã lập bản đồ.

Có những trọng số ở dạng Q4_K.

Có những trọng số ở dạng Q6_K.

Q4_K và Q6_K đều là các dạng **lượng tử hóa — cách đóng gói trọng số bằng ít bit hơn để giảm lượng dữ liệu phải lưu và di chuyển**.

Ta đã gặp Q4_K nhiều lần.

Ở P4, Q6_K bắt đầu trở nên bắt buộc vì các khối số thật của lớp không sử dụng chỉ một kiểu lượng tử hóa.

Cụ thể, khối số V của cơ chế chú ý và khối số `FFN-down` của lớp này dùng Q6_K.

Điều đó có nghĩa ArcLLM không thể nói:

> “Q4_K đã chạy được rồi, vậy cứ giả sử phần còn lại cũng giống thế.”

Muốn chạy lớp thật, hệ thực thi phải đọc và tính được **cả Q4_K lẫn Q6_K ở dạng đóng gói thật**.

Đây là lần đầu ta nhìn thấy một nguyên tắc sẽ lặp lại nhiều lần trong hành trình ArcLLM:

> **mô hình thật thường phá những giả định quá đẹp được hình thành từ một phép thử nhỏ.**

## Một lớp không chỉ là toán — còn là đường đi của dữ liệu

Giả sử phép A cho ra một kết quả đúng.

Phép B cũng đúng nếu ta đưa cho nó đầu vào đúng.

Nhưng trong lớp thật, kết quả của A chính là đầu vào của B.

Nếu A ghi dữ liệu sai chỗ, hoặc B đọc nhầm vùng nhớ — **vùng chứa dữ liệu** — thì cả hai chương trình GPU có thể đúng riêng lẻ mà hệ thống vẫn sai.

Ta có thể hình dung:

```text
Chương trình GPU A
   ↓
vùng nhớ X

Chương trình GPU B đáng lẽ đọc vùng X
nhưng lại đọc Y
   ↓
sai
```

Đây là lý do P4 có giá trị khác P3.

P3 chủ yếu hỏi:

> “Từng phép toán có đúng không?”

P4 thêm một câu hỏi mới:

> **“Các phép toán có được nối với nhau đúng không?”**

Ta bắt đầu kiểm tra cả **computation — phép tính** lẫn **dataflow — đường đi của dữ liệu**.

## 15 lần giao việc cho GPU trong một chuỗi thật

lớp `blk.0` của P4 được thực thi bằng:

```text
15 Vulkan dispatches
```

Nhắc lại, **lần giao việc cho GPU — một lần hệ thực thi giao một công việc tính toán cụ thể cho GPU**.

Có thể hình dung mỗi lần giao việc cho GPU là một công đoạn trong dây chuyền:

```text
lần giao việc 1
    ↓
lần giao việc 2
    ↓
lần giao việc 3
    ↓
...
    ↓
lần giao việc 15
```

Con số 15 không có nghĩa một lớp giải mã nói chung luôn phải có đúng 15 lần giao việc cho GPU.

Nó chỉ mô tả implementation P4 đã được kiểm tra.

Điều quan trọng hơn là cả 15 công việc này được ghi vào **một bộ lệnh — một danh sách lệnh GPU đã chuẩn bị trước**, rồi gửi xuống bằng:

```text
1 bộ lệnh
→ 1 lần submit
→ 1 lần fence wait
```

Nhắc lại:

- **submit**: đưa danh sách công việc vào hàng đợi GPU;
- **tín hiệu hoàn thành**: tín hiệu để CPU biết GPU đã thực hiện xong chuỗi công việc;
- **tín hiệu hoàn thành wait**: CPU chờ tín hiệu hoàn tất trước khi kiểm tra kết quả cuối.

Bức tranh bây giờ khác hẳn P3.

Không còn kiểu:

```text
chạy phép A
→ mang kết quả về CPU
→ kiểm tra

chạy phép B
→ mang kết quả về CPU
→ kiểm tra
```

P4 muốn cả chuỗi chạy liền mạch.

## Không “chạy về CPU hỏi ý kiến” giữa chừng

Một điều được khóa rất rõ trong P4 là:

> **không có vòng lặp tính toán trung gian quay ngược về CPU (vòng đi-về trung gian qua CPU).**

Đây là thuật ngữ chúng ta sẽ dùng từ đây về sau.

Ở cấp **tiêu chuẩn kỹ thuật đã khóa**, P4 ghi điều này dưới dạng:

> **zero intermediate host read/write**

**Host** ở đây là phía CPU và bộ nhớ mà chương trình trên CPU sử dụng trực tiếp.

**Intermediate — dữ liệu trung gian —** là kết quả đang nằm giữa đầu vào và đầu ra cuối cùng của lớp.

Nói đơn giản, P4 không cho phép đường thực thi:

> **GPU tạo dữ liệu trung gian → đưa về CPU để đọc hoặc sửa → rồi gửi xuống GPU trở lại trước khi tiếp tục.**

Đó chính là **vòng đi-về trung gian qua CPU — vòng lặp tính toán trung gian quay ngược về CPU** mà P4 muốn loại bỏ.

Dữ liệu đi theo hướng:

```text
GPU
 ↓
phép A
 ↓
dữ liệu trung gian
 ↓
phép B
 ↓
dữ liệu trung gian
 ↓
phép C
 ↓
...
 ↓
kết quả cuối lớp
```

thay vì:

```text
GPU
 ↓
CPU
 ↓
GPU
 ↓
CPU
 ↓
GPU
```

Điều này quan trọng vì nếu CPU chen vào giữa từng phép, ta chưa thật sự chứng minh được một lớp GPU-resident — **một lớp có dữ liệu trung gian được giữ ở phía GPU trong suốt chuỗi thực thi**.

P4 yêu cầu các intermediate — **kết quả tạm giữa các phép toán** — tiếp tục cư trú ở phía GPU.

Chỉ khi cả lớp hoàn thành, kết quả cuối mới được đem ra để so với CPU reference.

## Nhưng làm sao biết cả lớp đúng?

Ta quay lại nguyên tắc của P3.

GPU không được tự chấm bài cho chính mình.

ArcLLM có một **independent CPU reference — cách tính tham chiếu độc lập trên CPU** cho cả lớp.

Cùng một đầu vào.

Cùng trọng số thật.

Một bên tính bằng chuỗi Vulkan trên GPU.

Một bên tính độc lập trên CPU.

Sau đó so hai đầu ra.

Nhưng lần này chỉ nhìn một con số chênh lệch là chưa đủ, bởi đầu ra của lớp là cả một dãy giá trị.

P4 dùng hai thước đo:

**max_abs** và **RMSE**.

Tên hơi khó, nhưng ý nghĩa có thể hiểu rất trực tiếp.

## Sai số lớn nhất (max_abs): điểm lệch nhiều nhất là bao nhiêu?

Giả sử CPU cho:

```text
[1,00, 2,00, 3,00]
```

GPU cho:

```text
[1,01, 1,98, 3,02]
```

Sai khác tuyệt đối từng vị trí là:

```text
|1,01 - 1,00| = 0,01
|1,98 - 2,00| = 0,02
|3,02 - 3,00| = 0,02
```

Vậy sai khác lớn nhất là:

```text
max_abs = 0,02
```

**max_abs — maximum absolute error — sai số tuyệt đối lớn nhất** trả lời câu hỏi:

> “Trong tất cả các giá trị, điểm tệ nhất lệch bao nhiêu?”

Nó rất hữu ích để bắt một phần tử bị sai nổi bật.

Nhưng nó chỉ nhìn phần tử tệ nhất.

Ta cần thêm một góc nhìn khác.

## Sai số tổng thể (RMSE): nhìn cả dãy thay vì một điểm

**RMSE — Root Mean Square Error — căn trung bình bình phương sai số** nghe khá toán học.

Ta vẫn dùng ví dụ vừa rồi.

Sai số là:

```text
0,01
0,02
0,02
```

Bình phương chúng:

```text
0,01² = 0,0001
0,02² = 0,0004
0,02² = 0,0004
```

Lấy trung bình:

```text
(0,0001 + 0,0004 + 0,0004) / 3
= 0,0003
```

Rồi lấy căn:

```text
RMSE = √0,0003
≈ 0,0173
```

Không cần ghi nhớ công thức.

Chỉ cần nhớ ý nghĩa:

> **RMSE cho ta một con số mô tả mức sai lệch điển hình của cả dãy, đồng thời phạt những sai lệch lớn mạnh hơn vì chúng bị bình phương.**

Vì vậy hai thước đo bổ sung cho nhau:

```text
max_abs
→ điểm tệ nhất sai bao nhiêu?

RMSE
→ toàn bộ dãy nhìn chung sai bao nhiêu?
```

## Khóa tiêu chuẩn trước khi xem đáp án

P4 không chạy xong rồi mới quyết định:

> “Sai khoảng này chắc là chấp nhận được.”

Hai cổng đã được **freeze — khóa trước khi xem outcome**:

```text
max_abs <= 2e-2
RMSE    <= 5e-3
```

Ta đổi sang cách viết quen hơn:

```text
max_abs <= 0,02
RMSE    <= 0,005
```

Nghĩa là lớp chỉ được ĐẠT (PASS) nếu:

- không có phần tử nào lệch quá 0,02 theo max_abs;
- sai số tổng thể theo RMSE không vượt 0,005.

Tại sao phải khóa trước?

Bởi nếu xem kết quả rồi mới đặt tiêu chuẩn, ta rất dễ làm điều này:

```text
kết quả sai 0,03
       ↓
"vậy ngưỡng 0,04 chắc hợp lý"
       ↓
PASS
```

Khi đó gate không còn kiểm tra giả thuyết nữa.

Ta chỉ đang chỉnh luật để kết quả mình muốn thắng.

## Kết quả P4

Khi lớp thật `blk.0` chạy xong, kết quả đo được là:

```text
max_abs
= 0,0005810260773

ngưỡng
<= 0,02
```

và:

```text
RMSE
= 0,00003027076833

ngưỡng
<= 0,005
```

Cả hai đều nằm trong cổng đã khóa trước.

Vì vậy P4 ĐẠT (PASS).

Ta nên đọc kết quả bằng câu tiếng Việt trước khi nhìn vào nhiều số:

> **Đầu ra của lớp giải mã chạy trên GPU đủ gần với đầu ra của cách tính tham chiếu độc lập trên CPU theo cả hai tiêu chuẩn đã định trước.**

Không cần biến những con số nhỏ này thành tuyên bố lớn hơn.

P4 không chứng minh GPU “chính xác tuyệt đối”.

Nó chứng minh sai số nằm trong **ngưỡng đã đăng ký trước**.

## Đây là bước tiến lớn hơn P3 ở đâu?

P3 giống như kiểm tra từng nhạc công có chơi đúng phần của mình hay không.

P4 giống như cho cả nhóm chơi một đoạn nhạc cùng nhau.

Một người chơi đúng riêng không bảo đảm cả nhóm vào đúng nhịp.

Ở P4, ta đồng thời kiểm tra:

```text
chương trình GPU đúng
+
trọng số thật đúng
+
Q4_K đúng
+
Q6_K đúng
+
thứ tự phép toán đúng
+
vùng nhớ nối đúng
+
residual đúng
+
intermediate nằm đúng nơi
+
cả chuỗi cuối cùng đúng
```

Dĩ nhiên một ĐẠT (PASS) không chứng minh từng dòng implementation là hoàn hảo.

Nhưng nó loại bỏ được một lớp rủi ro lớn hơn nhiều so với P3.

Ta không còn chỉ có một bộ sưu tập phép tính nền tảng tốt.

Ta đã có **một lớp giải mã thật hoạt động end-to-end trong phạm vi lớp**.

## Một lần dừng trước thực thi — và lần đầu xuất hiện “sổ nghiên cứu”

P4 cũng có một lần dừng trước khi GPU thực sự chạy lớp.

Trong lần dừng đó xuất hiện một tên tệp mà người đọc chưa gặp trước đây: `lineage.md`.

Đây là lúc cần tách thật rõ **quản trị nghiên cứu** khỏi **thực thi mô hình**.

`lineage.md` chỉ là **một tệp văn bản dùng như sổ lịch sử nghiên cứu**. Dự án ghi vào đó những mốc như: câu hỏi đang kiểm tra là gì, evidence nào đã có, quyết định nào được đưa ra và bước tiếp theo là gì.

Nó **không tham gia vào phép tính GPU**.

Nó không chứa khối số.

Nó không được Vulkan đọc để chạy lớp giải mã.

Nó xuất hiện ở đây chỉ vì trước khi cho phép package chạy, một bài QA — **kiểm tra chất lượng của gói thực thi** — có bước kiểm tra tính nhất quán của tài liệu nghiên cứu này.

Lần đó, bài kiểm tra package đọc `lineage.md` bằng encoding mặc định của Windows. Trong tệp có một ký tự dấu gạch dài Unicode, khiến phép so sánh text bị lỗi.

Kết quả thực tế là:

```text
QA của package
→ FAIL

shader compile
→ chưa chạy

native build
→ chưa chạy

thực thi lớp bằng Vulkan
→ chưa chạy
```

Điều này không thể được ghi là:

```text
P4 scientific FAIL
```

Bởi lớp giải mã chưa hề được thực thi.

Lỗi được sửa ở lớp **đóng gói và kiểm tra** bằng cách làm cách mã hóa rõ ràng hơn. Tiêu chuẩn khoa học đã khóa không thay đổi. Các chương trình GPU, trọng số và ngưỡng sai số cũng không được sửa để chiều theo kết quả.

Sau đó P4 mới được chạy thật và ĐẠT (PASS).

Chi tiết `lineage.md` được giữ lại trong sách vì nó mở ra một lớp câu chuyện khác: **kết quả khoa học không chỉ cần được tạo ra, mà còn phải được ghi nhận sao cho sau này có thể truy lại được ta đã biết gì ở từng thời điểm**.

Tạm thời người đọc chỉ cần nhớ:

> **`lineage.md` là sổ ghi lịch sử nghiên cứu, không phải một phần của hệ thực thi.**

Cách quản trị sâu hơn — gồm các chế độ dùng để loại nhanh giả thuyết, đào sâu hoặc chuyển cách nghiên cứu — sẽ chỉ xuất hiện về sau, khi chính câu chuyện ArcLLM buộc chúng ta phải dùng chúng. Ở đây chưa cần mang toàn bộ hệ quản trị vào một chương đang nói về lớp giải mã.

Điểm cần giữ lúc này chỉ là:

> **Phải biết thất bại xảy ra ở tầng nào trước khi quyết định nó phủ định điều gì.**

## P4 ĐẠT cho phép kết luận gì?

Tóm tắt P4:

```text
1 lớp giải mã thật
→ blk.0 của Qwen2

15 Vulkan dispatches
→ 15 công việc tính toán GPU

1 bộ lệnh
→ một danh sách lệnh GPU

1 submit
→ gửi cả chuỗi xuống hàng đợi tính toán một lần

1 fence wait
→ chờ GPU hoàn tất cả chuỗi

Q4_K + Q6_K packed weights
→ dùng trọng số thật vẫn ở dạng lượng tử hóa đóng gói

resident intermediates
→ dữ liệu trung gian tiếp tục nằm ở phía GPU

zero intermediate host read/write
→ không có intermediate host round-trip
→ không quay ngược dữ liệu trung gian về CPU giữa các phép

max_abs = 0,0005810260773
→ PASS dưới ngưỡng 0,02

RMSE = 0,00003027076833
→ PASS dưới ngưỡng 0,005
```

Nhưng P4 vẫn **chưa** chứng minh:

- toàn bộ decoder chạy đúng;
- tất cả lớp đều đúng;
- mô hình sinh token;
- hệ thực thi nhanh;
- ArcLLM tốt hơn một hệ thực thi khác.

P4 chỉ cho phép ta nói:

> **Một lớp giải mã thật, dùng trọng số thật Q4_K và Q6_K, đã chạy trọn chuỗi trên Vulkan với các kết quả trung gian giữ ở phía GPU và đầu ra cuối vượt qua hai cổng sai số đã khóa.**

Đủ để đi tiếp.

Không hơn.

## Từ một lớp tới toàn bộ khối giải mã

Bây giờ ta gặp một câu hỏi rất tự nhiên.

Một lớp đã chạy được.

Nếu mô hình có nhiều lớp nối tiếp nhau, liệu ta có thể giữ toàn bộ phần decoder trong bộ nhớ và cho tín hiệu đi xuyên qua hết chuỗi mà không phải liên tục quay về CPU hay không?

Đây là bước nhảy tiếp theo.

Không còn:

```text
một phép tính nền tảng
```

cũng không còn:

```text
một lớp
```

mà là:

```text
phép nhúng
   ↓
lớp
   ↓
lớp
   ↓
lớp
   ↓
...
   ↓
norm
   ↓
đầu ra cuối decoder
```

P5 sẽ chuyển câu hỏi từ **“một căn phòng hoạt động chưa?”** sang **“cả tòa nhà có thể được dựng và giữ hoạt động cùng lúc không?”**

### Nhớ 3 điều

1. **phép tính nền tảng ĐẠT (PASS) chưa bảo đảm composition ĐẠT (PASS).** Các phép toán đúng riêng lẻ vẫn có thể sai khi ghép vì thứ tự, vùng nhớ hoặc đường đi dữ liệu.
2. **P4 giữ intermediate — dữ liệu trung gian — ở phía GPU suốt lớp.** Không có vòng CPU chen vào giữa để “cứu” kết quả.
3. **P4 chỉ ĐẠT về tính đúng của một lớp, chưa phải của toàn bộ mô hình.** Một lớp thật đã vượt các tiêu chuẩn đã khóa; toàn bộ chuỗi lớp giải mã vẫn là câu hỏi của bước tiếp theo.

**Chương 6 — giữ toàn bộ khối giải mã sẵn trong bộ nhớ**

Ta đã xây được một căn phòng hoàn chỉnh.

Bây giờ ArcLLM sẽ thử giữ cả tòa nhà trong bộ nhớ.
