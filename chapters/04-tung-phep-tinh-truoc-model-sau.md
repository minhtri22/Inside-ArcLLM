# Chương 4 — Từng phép tính trước, model sau

> **Mức đọc: Đi sâu**
>
> **Bản đồ xuyên suốt — đang mở: RMSNorm / attention / FFN**
>
> ```text
> văn bản → token → tensor
>                     ↓
>          model / parameters
>                     ↓
>                  runtime
>                     ↓
>            CPU / GPU / bộ nhớ
>                     ↓
>      RMSNorm / attention / FFN
>                     ↓
>              decoder layer
>                     ↓
>             nhiều decoder layer
>                     ↓
>          KV cache / sinh token
>                     ↓
>        benchmark / tối ưu
>                     ↓
> representation / lifecycle / kiến trúc runtime
> ```
>
> ▶ **Đang mở ở chương này:** RMSNorm / attention / FFN.


> **Câu hỏi của chương:** Trước khi ghép cả model lại với nhau, làm thế nào biết GPU đang tính đúng từng phép toán nhỏ bên trong nó?

Ở cuối Chương 3, ArcLLM đã dựng được phần nền của một “nhà máy GPU”.

Runtime đã mở được thiết bị Vulkan, có hàng đợi để giao việc, có vùng nhớ giữ dữ liệu model, có vùng nhớ tạm để làm việc, và quan trọng nhất: ArcLLM đã gửi một lệnh thật xuống GPU rồi xác nhận GPU thực hiện nó.

Nhưng có một khoảng cách rất lớn giữa:

> “GPU nhận được lệnh của tôi”

và:

> “GPU đang tính đúng một model AI.”

P2 mới chứng minh điều đầu tiên.

P3 bắt đầu đi vào điều thứ hai.

Một lựa chọn dễ hiểu nhưng nguy hiểm lúc này là ghép luôn cả model, đưa một câu hỏi vào rồi xem nó có trả lời được hay không.

ArcLLM không làm vậy.

Thay vào đó, dự án tách model thành những phép tính nhỏ hơn, kiểm tra từng phép một, rồi chỉ khi những viên gạch đó đủ đáng tin mới bắt đầu xây cả bức tường.

## Model lớn nhưng được tạo thành từ những việc nhỏ hơn

Một model ngôn ngữ có thể chứa hàng tỷ trọng số và rất nhiều lớp.

Nghe như một hệ thống không thể hiểu nổi.

Nhưng khi runtime thực thi nó, công việc cuối cùng vẫn được phân rã thành những phép toán cụ thể.

Ví dụ, ở P3 ta gặp những nhóm như:

- chuẩn hóa một dãy số;
- nhân dữ liệu với trọng số;
- thêm thông tin vị trí của token;
- biến một dãy điểm số thành những tỷ lệ có ý nghĩa;
- cho các token “chú ý” tới thông tin trước đó;
- đi qua một nhánh xử lý rồi cộng kết quả trở lại.

Tên kỹ thuật của chúng sẽ xuất hiện ngay sau đây.

Nhưng trước hết hãy giữ một hình dung đơn giản:

```text
MODEL

không phải một phép tính khổng lồ

mà là

phép A
  ↓
phép B
  ↓
phép C
  ↓
phép D
  ↓
...
```

P3 hỏi:

> **Nếu lấy từng phép riêng ra, GPU có cho kết quả đủ gần với một cách tính tham chiếu độc lập trên CPU hay không?**

Đó là câu hỏi khoa học của bước này.

## Kernel là gì?

Ở Chương 3, GPU mới được yêu cầu điền một giá trị thử vào vùng nhớ.

Bây giờ GPU phải thực hiện những phép toán có ý nghĩa đối với model.

Một chương trình nhỏ chạy trên GPU để thực hiện một loại công việc cụ thể thường được gọi là **kernel — chương trình tính toán nhỏ chạy trên GPU**.

Ví dụ, ta có thể có một kernel chuyên chuẩn hóa dữ liệu.

Một kernel khác chuyên nhân dữ liệu đầu vào với trọng số.

Một kernel khác nữa xử lý attention.

Có thể hình dung:

```text
CPU / ArcLLM
      ↓
"hãy thực hiện phép này"
      ↓
Vulkan
      ↓
GPU kernel
      ↓
kết quả
```

Trong Vulkan, những chương trình tính toán này được viết dưới dạng **compute shader — chương trình tính toán dành cho GPU** rồi được biên dịch sang dạng GPU có thể thực thi.

P3 sử dụng một compiler — **trình biên dịch** — đã được khóa phiên bản: `glslang 16.5.0`.

Việc khóa phiên bản không nhằm làm câu chuyện phức tạp hơn.

Nó nhằm giữ nguyên câu hỏi thí nghiệm.

Nếu hôm nay shader được biên dịch bằng một phiên bản, ngày mai bằng phiên bản khác, rồi kết quả thay đổi, ta vừa thêm một biến mà mình không kiểm soát.

P3 vì vậy cũng xác nhận nguồn gốc và đúng phiên bản compiler trước khi tin vào kết quả phía sau.

## CPU làm “đáp án tham chiếu”

Có một vấn đề thú vị.

Nếu ta vừa tự viết runtime vừa tự viết kernel GPU, làm sao biết kết quả GPU là đúng?

Không thể hỏi chính kernel đó:

> “Anh có tính đúng không?”

ArcLLM dùng một cách rất phổ biến trong kỹ thuật số: tạo một **CPU reference — cách tính tham chiếu độc lập trên CPU**.

Hãy tưởng tượng học sinh làm một phép tính bằng máy tính bỏ túi, còn giáo viên đã có một cách giải độc lập.

Sau đó ta so hai kết quả.

```text
CPU reference
      ↓
kết quả A

GPU kernel
      ↓
kết quả B

A và B đủ gần nhau?
      ↓
PASS / FAIL
```

Tại sao lại nói “đủ gần” mà không phải “giống từng chữ số”?

Vì nhiều phép tính với số thực có thể tạo ra sai khác rất nhỏ tùy cách phần cứng thực hiện và thứ tự tính toán.

Ví dụ:

```text
CPU: 1,234500
GPU: 1,234501
```

Hai con số không hoàn toàn giống nhau ở từng chữ số, nhưng chênh lệch chỉ:

```text
1,234501 - 1,234500
= 0,000001
```

Một sai khác rất nhỏ như vậy có thể nằm trong giới hạn đã được xác định trước.

Điều quan trọng là giới hạn đó phải được khóa **trước khi nhìn kết quả**, chứ không phải thấy kết quả sai bao nhiêu rồi mới nới tiêu chuẩn cho vừa.

## Viên gạch đầu tiên: RMSNorm

Phép đầu tiên đáng làm quen là **RMSNorm — một phép chuẩn hóa giúp giữ độ lớn của tín hiệu ở mức phù hợp**.

Ta chưa cần đi vào công thức đầy đủ.

Hãy hình dung một lớp nhận một dãy số:

```text
[ rất nhỏ, vừa, rất lớn, ... ]
```

Nếu độ lớn của tín hiệu thay đổi quá tự do khi đi qua nhiều lớp, việc tính toán về sau trở nên khó kiểm soát.

RMSNorm đo độ lớn chung của dãy rồi điều chỉnh nó trước khi bước tiếp.

Có thể hình dung như chỉnh âm lượng:

```text
tín hiệu đầu vào
      ↓
đo mức chung
      ↓
điều chỉnh
      ↓
tín hiệu có thang phù hợp hơn
```

P3 không chỉ thử RMSNorm với một dãy số tưởng tượng.

Nó sử dụng trọng số thật:

`blk.0.attn_norm.weight`

từ model đã được khóa.

GPU tính RMSNorm.

CPU cũng tính độc lập.

Hai kết quả được so với nhau.

Gate — **cổng kiểm tra** — đạt PASS.

Đây là lần đầu trong chuỗi P3 mà một phép toán thật của model cùng trọng số thật đi qua Vulkan và vượt qua kiểm tra số học.

## Q4_K: không mở hộp trước rồi mới tính

Chương 2 đã dành khá nhiều thời gian cho Q4_K.

Ta biết đây là một dạng **quantization — cách lưu trọng số bằng ít bit hơn**, và ArcLLM cố tình giữ dữ liệu ở dạng đóng gói thay vì bung toàn bộ sang F32.

P3 bây giờ phải trả lời câu hỏi khó hơn:

> Giữ Q4_K đóng gói thì tốt cho bộ nhớ, nhưng GPU có thể tính trực tiếp từ dạng đó không?

Kernel P3 sử dụng trọng số thật:

`blk.0.attn_q.weight`

và đọc trực tiếp dữ liệu Q4_K đang ở dạng packed — **vẫn đóng gói**.

Không có bước:

```text
Q4_K
  ↓
bung toàn bộ thành F32
  ↓
GPU mới tính
```

Đường P3 muốn kiểm tra là:

```text
Q4_K packed
     ↓
GPU kernel đọc trực tiếp
     ↓
phép nhân
     ↓
kết quả
```

Đây là một ranh giới rất quan trọng.

Nếu P1 chỉ chứng minh:

> “Ta nhìn thấy được Q4_K trong file.”

thì P3 bắt đầu chứng minh:

> “Ta có thể dùng chính dạng Q4_K đóng gói đó trong một phép toán GPU thật.”

Phần này bao gồm **matvec/GEMM — các phép nhân giữa vector hoặc ma trận với trọng số**.

Không cần học đại số tuyến tính ở đây.

Ta chỉ cần hình dung:

```text
dữ liệu đang đi qua model
        ×
trọng số model đã học
        ↓
dữ liệu mới
```

Đây là một trong những công việc xuất hiện với số lượng rất lớn trong model.

## RoPE: cho token biết vị trí

Một model không chỉ cần biết token nào xuất hiện.

Nó còn cần biết chúng nằm ở đâu trong chuỗi.

Hai câu:

> “chó cắn người”

và:

> “người cắn chó”

có cùng ba từ nhưng thứ tự tạo ra ý nghĩa rất khác.

Một cơ chế được model này sử dụng là **RoPE — Rotary Position Embedding**, có thể hiểu ở mức đầu tiên là **cách đưa thông tin vị trí vào dữ liệu bằng một phép biến đổi dạng xoay**.

Ta chưa cần hiểu hình học phía sau chữ “xoay”.

Hiện tại chỉ cần nhớ:

```text
nội dung token
      +
token đang ở vị trí nào
      ↓
tín hiệu được biến đổi
```

P3 kiểm tra phép RoPE trên GPU với một CPU reference độc lập.

Nếu GPU xử lý vị trí sai, model về sau có thể hiểu sai quan hệ thứ tự ngay cả khi các phép nhân khác đều đúng.

Vì vậy RoPE cũng phải được chứng minh riêng trước khi ghép thành layer.

## Softmax: từ điểm số thành tỷ lệ

Một phép toán khác là **softmax — phép biến một nhóm điểm số thành những giá trị dương có tổng bằng 1**.

Ví dụ đơn giản, giả sử sau một số bước ta có ba giá trị cuối:

```text
0,2
0,3
0,5
```

Tổng là:

```text
0,2 + 0,3 + 0,5 = 1,0
```

Ta có thể hình dung chúng như:

```text
20%
30%
50%
```

Softmax thật không đơn giản chỉ là chia cho tổng như ví dụ trên; nó còn dùng hàm mũ để xử lý các điểm số đầu vào.

Nhưng ý nghĩa mà người đọc cần giữ ở đây là:

> **Softmax giúp chuyển các điểm số so sánh thành một phân bố trọng số có thể dùng để quyết định phần thông tin nào được chú ý nhiều hơn.**

P3 chưa cần giải cả attention của model để kiểm tra softmax.

Nó có thể kiểm tra phép toán này như một viên gạch riêng.

GPU tính.

CPU reference tính.

Rồi hai bên được so.

## Attention: token nhìn lại thông tin liên quan

Bây giờ ta tới một thuật ngữ nổi tiếng hơn: **attention — cơ chế cho phép model cân nhắc những phần thông tin khác nhau khi xử lý token hiện tại**.

Hãy lấy một câu đơn giản:

> “Nam đặt chiếc cốc lên bàn vì **nó** bị ướt.”

Từ “nó” có thể cần liên hệ với một phần xuất hiện trước đó.

Model không làm việc này bằng cách “hiểu như con người” theo đúng nghĩa đời thường. Bên dưới vẫn là những phép toán tạo điểm số, chuẩn hóa chúng rồi kết hợp thông tin.

P3 sử dụng một bài kiểm tra **bounded GQA attention**.

Ta tách cụm này ra:

- **attention**: cơ chế kết hợp thông tin theo mức liên quan;
- **GQA — Grouped Query Attention**: một cách tổ chức attention mà model mục tiêu sử dụng;
- **bounded**: P3 chỉ kiểm tra trong một phạm vi nhỏ, có kiểm soát, chưa phải toàn bộ model.

Từ “bounded” rất quan trọng.

Một phép attention nhỏ PASS không có nghĩa:

> “Attention của toàn model chắc chắn đúng.”

Nó chỉ cho phép nói:

> **Cơ chế đã được hiện thực hóa đủ đúng trong phạm vi test đã khóa để được dùng làm viên gạch tiếp theo.**

## SwiGLU và residual: biến đổi rồi cộng trở lại

Một decoder layer còn có một nhánh thường được gọi là FFN — **Feed-Forward Network, nhánh biến đổi tín hiệu sau attention**.

P3 kiểm tra một phần quan trọng của nhánh này bằng **SwiGLU**.

Ta chưa cần nhớ công thức.

Có thể hình dung SwiGLU như một cánh cổng:

```text
tín hiệu
   ↓
hai nhánh biến đổi
   ↓
một nhánh điều tiết nhánh còn lại
   ↓
kết quả
```

Sau đó model còn sử dụng **residual — đường cộng tắt**, tức kết quả mới được cộng trở lại với tín hiệu cũ.

Ẩn dụ đơn giản:

```text
tín hiệu ban đầu ───────────────────┐
                                   │
          ↓                        │
     biến đổi                      │
          ↓                        │
     kết quả mới                   │
          ↓                        │
          └──────── cộng lại ──────┘
```

Residual giúp thông tin có một đường đi trực tiếp qua các lớp thay vì mọi thứ đều phải bị thay thế hoàn toàn ở mỗi bước.

P3 kiểm tra cả nhóm **SwiGLU + residual** như một primitive — **phép toán nền tảng** — trước khi nó được ghép vào decoder layer hoàn chỉnh.

## Bảy cổng, nhưng chưa có một model

Tổng cộng P3 đóng **bảy Vulkan kernel gates — bảy cổng kiểm tra cho các phép tính GPU**.

Chúng bao phủ các nhóm phép toán cần thiết ở bước này:

```text
RMSNorm
→ chuẩn hóa tín hiệu

packed Q4_K matvec/GEMM
→ nhân dữ liệu với trọng số Q4_K vẫn đóng gói

RoPE
→ đưa thông tin vị trí vào tín hiệu

softmax
→ biến điểm số thành phân bố trọng số

bounded GQA attention
→ attention GQA trong phạm vi kiểm tra nhỏ

SwiGLU + residual
→ biến đổi qua nhánh FFN rồi cộng đường tắt
```

Cả bảy gate đều PASS khi so với **independent CPU references — cách tính tham chiếu độc lập trên CPU**.

Điều này quan trọng.

Nhưng nó vẫn chưa cho phép nói:

> “ArcLLM đã chạy được model.”

P3 chỉ nói:

> **Những viên gạch số học cần thiết ở phạm vi P3 đã vượt qua kiểm tra riêng lẻ.**

Một đống gạch tốt chưa phải một ngôi nhà.

Bước tiếp theo phải kiểm tra xem khi ghép chúng theo đúng thứ tự của model, toàn bộ một layer có còn đúng hay không.

## Hai lần dừng trước đó vẫn không phải kernel FAIL

P3 cũng từng có những lần chưa tới được phép thử thật.

Một lần package thiếu file cấu hình.

Một lần khác shader đã compile nhưng quá trình build dừng vì những source cần thiết từ GGUF/tensor store chưa được đóng gói cùng.

Trong cả hai trường hợp:

```text
kernel chưa chạy
      ↓
không có output GPU
      ↓
không có so sánh CPU ↔ GPU
      ↓
không có verdict khoa học
```

Sau khi phần đóng gói được sửa mà không thay đổi contract số học của P3, phép thử thật mới chạy.

Khi đó bảy gate mới được adjudicate — **đánh giá theo tiêu chuẩn đã khóa** — là PASS.

Đây là lần thứ hai cuốn sách gặp cùng một nguyên tắc, và nó đáng để lặp lại:

> **Không phải mọi chữ đỏ trên màn hình đều là một giả thuyết thất bại.**

Nếu chưa thực sự chạm tới câu hỏi nghiên cứu thì ta chỉ biết hệ thống chưa chạy tới nơi cần đo.

## P3 đã cho phép chúng ta tin điều gì?

Tới cuối P3, ArcLLM có thể nói:

```text
glslang 16.5.0
→ trình biên dịch shader đã được khóa và xác minh nguồn gốc

7 Vulkan kernel gates
→ 7 cổng kiểm tra phép toán GPU
→ tất cả PASS so với CPU reference độc lập

RMSNorm
→ dùng trọng số thật blk.0.attn_norm.weight

Q4_K GPU path
→ dùng trực tiếp trọng số thật blk.0.attn_q.weight
→ vẫn ở dạng packed, không bung toàn bộ trước

RoPE / softmax / bounded GQA attention
→ các viên gạch attention đã qua gate riêng

SwiGLU + residual
→ viên gạch FFN và đường cộng tắt đã qua gate
```

Nhưng vẫn phải nhắc ngay điều P3 **chưa chứng minh**:

- chưa có một decoder layer hoàn chỉnh;
- chưa chạy 28 layer;
- chưa sinh token;
- chưa chứng minh tốc độ;
- chưa chứng minh model end-to-end đúng.

P3 PASS chỉ mở quyền đi sang câu hỏi mới:

> **Nếu từng viên gạch đều đúng riêng lẻ, khi ghép chúng thành một decoder layer thật với trọng số thật, kết quả cuối layer có còn đúng không?**

Đó là P4.

### Nhớ 3 điều

1. **Kernel — chương trình tính toán nhỏ trên GPU — phải được kiểm tra riêng trước khi được tin tưởng trong một model lớn.**
2. **CPU reference — cách tính tham chiếu độc lập trên CPU — đóng vai trò “đáp án” để kiểm tra GPU, và ngưỡng sai số phải được khóa trước khi nhìn kết quả.**
3. **P3 PASS là PASS của các primitive — những phép toán nền tảng — chứ chưa phải PASS của decoder layer hay toàn model.**

**Chương 5 — Một decoder layer hoàn chỉnh**

Ở P3, ta đã đặt từng viên gạch lên bàn và thử riêng từng viên.

P4 sẽ làm điều nguy hiểm hơn nhiều:

**ghép chúng lại theo đúng đường đi của một decoder layer thật, sử dụng trọng số thật, giữ các kết quả trung gian trên GPU, rồi kiểm tra xem kết quả cuối cùng còn khớp với CPU hay không.**
