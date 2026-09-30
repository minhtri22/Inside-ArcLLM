# Chương 11 — Từ một thất bại tới câu hỏi đúng hơn

> **Mức đọc: Nghiên cứu**
>
> **Bạn đang ở bước nào của hành trình nghiên cứu?**
>
> ```text
> Thất bại
>    ↓
> tách chi phí theo từng vùng
>    ↓
> [ đặt giả thuyết có thể kiểm tra ]
>    ↓
> chưa vội tối ưu
> ```


> **Câu hỏi của chương:** Khi một kiến trúc đã bị bằng chứng đo lường buộc phải dừng lại, điều gì đủ mạnh để cho phép ta mở một kiến trúc kế tiếp?

Chương 10 kết thúc bằng một kết luận rất rõ:

```text
FEASIBLE_NO_DEMONSTRATED_ADVANTAGE
```

Dịch cẩn thận:

> **ArcLLM đã chứng minh được rằng nó có thể chạy mô hình thật, nhưng chưa chứng minh được một lợi thế thực tế so với llama.cpp trong các điều kiện đã kiểm tra.**

Đó không phải là một lỗi kỹ thuật.

Không phải thí nghiệm chưa chạy xong.

Không phải vì thiếu dữ liệu.

Q3 đã hoàn thành đủ 40/40 lần đo mới.

Hai phiên chạy độc lập cùng đi tới một kết luận.

Vì vậy quy tắc dừng đã khóa trước buộc ta phải:

> **đóng kiến trúc ArcLLM hiện tại.**

Đây là một thời điểm rất dễ đi sai hướng.

Ta biết hệ thống đang chậm hơn đối chứng rất nhiều.

Ta cũng biết còn vô số ý tưởng có thể thử.

AI thậm chí có thể tiếp tục đề xuất hàng chục biến thể chương trình GPU, khối xử lý, cách gộp phép tính hay cách lập lịch khác nhau chỉ trong vài phút.

Nhưng nếu cứ tiếp tục thử cho tới khi xuất hiện một con số đẹp, ta không còn nghiên cứu kiến trúc nữa.

Ta đang cố cứu kiến trúc cũ.

Câu hỏi vì vậy phải thay đổi.

Không còn:

> “Tối ưu thêm chỗ nào?”

Mà là:

> **“Có một cơ chế mới đủ rõ ràng để biện minh cho một kiến trúc kế tiếp hay không?”**

## Kiến trúc kế tiếp là gì?

Từ đây xuất hiện một ký hiệu mới:

> **Successor Architecture (SA) — kiến trúc kế tiếp.**

`SA` không đơn giản có nghĩa:

```text
ArcLLM v1
↓
ArcLLM v2
```

chỉ vì ta muốn phiên bản sau tốt hơn.

Một kiến trúc kế tiếp chỉ được phép mở khi:

```text
kiến trúc cũ đã bị đóng
↓
bằng chứng cho thấy một vấn đề cụ thể
↓
ta có một cơ chế mới có thể tác động trực tiếp vào vấn đề đó
↓
cơ chế ấy có thể bị kiểm tra và bác bỏ
```

Vì vậy:

```text
SA-H1
```

có nghĩa:

> **Successor Architecture — Hypothesis 1**, tức **giả thuyết số 1 cho kiến trúc kế tiếp**.

Còn:

```text
SA0
```

là:

> **bước số 0 của nghiên cứu kiến trúc kế tiếp**.

Ở SA0 chưa viết chương trình GPU mới.

Chưa chạy phép đo so sánh để tìm mức tăng tốc.

Chưa thay đổi hệ thực thi.

Ta chỉ hỏi:

> **Giả thuyết này có đủ cơ sở về cơ chế và phần cứng để đáng tiếp tục nghiên cứu hay không?**

Đó là một ranh giới rất quan trọng.

`SA-H1` mới chỉ là một giả thuyết.

Chưa phải một kiến trúc đã được chấp nhận.

## Trước hết phải hiểu đúng con số 469

Trong đường sinh từng token mới của ArcLLM có một con số rất dễ gây hiểu nhầm:

```text
469 dispatch / token
```

`Dispatch` có thể hiểu là:

> **một công việc tính toán được giao cho GPU thực hiện.**

Nhìn thấy 469, ta rất dễ nghĩ:

> “ArcLLM chậm vì CPU phải gọi GPU 469 lần.”

Nhưng Q2/Q3 đã xác nhận:

```text
1 submit / decode token
```

`Submit` ở đây là:

> **một lần CPU gửi cả chuỗi công việc đã chuẩn bị xuống GPU.**

Vì vậy thực tế gần hơn với:

```text
CPU
↓
gửi một chuỗi lệnh

GPU
↓
dispatch 1
dispatch 2
dispatch 3
...
dispatch 469
```

chứ không phải:

```text
CPU gọi GPU
↓
GPU chạy một việc
↓
CPU gọi lại
↓
GPU chạy tiếp
...
469 lần
```

Điều này lập tức loại bỏ một lời giải thích quá đơn giản:

> **469 lần giao việc cho GPU không có nghĩa ArcLLM đang trả chi phí 469 lần CPU giao việc cho GPU.**

Nếu muốn hiểu vì sao giai đoạn sinh token chậm, ta phải nhìn tiếp:

> **469 công việc đó thực sự đang làm gì?**

## Gần một nửa đồ thị sinh token là các phép nhân lớn

Mỗi token mới đi qua 28 decoder lớp.

Trong mỗi lớp có các bước như:

```text
RMSNorm

Q projection
K projection
V projection

RoPE
KV store
attention

O projection

FFN norm

gate projection
up projection
SwiGLU
down projection
```

Không cần thuộc tất cả tên này.

Điều cần chú ý là bảy phép chiếu:

```text
Q
K
V
O
gate
up
down
```

Mỗi phép chiếu chủ yếu là một phép nhân giữa dữ liệu đầu vào và một ma trận trọng số lớn.

Trong machine learning, loại phép tính này thường được gọi bằng thuật ngữ:

> **GEMM — phép nhân ma trận tổng quát (General Matrix Multiplication).**

Có:

```text
7 phép / layer
×
28 layer
=
196 phép GEMM / token
```

Trong tổng số 469 lần giao việc cho GPU:

```text
196
```

thuộc riêng họ công việc này.

Con số đó chưa chứng minh:

> “GEMM là nguyên nhân của toàn bộ hiệu năng khoảng cách.”

Nhưng nó cho ta một vùng đủ lớn và đủ cụ thể để bắt đầu kiểm tra.

## Xử lý đầu vào và sinh token đang được đối xử rất khác nhau

Ở Chương 8, P7 đã tối ưu khá sâu đường **giai đoạn xử lý đầu vào — giai đoạn mô hình xử lý toàn bộ prompt đầu vào**.

giai đoạn xử lý đầu vào đã có những đường tính toán chuyên biệt như:

```text
Q/K/V/O được chia tile
gate + up được gộp
FFN-down có đường tính toán riêng
```

Nhưng khi chuyển sang **giai đoạn sinh token — giai đoạn mỗi lần chỉ sinh thêm một token**, kiến trúc vẫn chủ yếu sử dụng đường GEMM dùng chung cho trường hợp batch bằng 1.

`Batch-1` nghĩa rất đơn giản:

> **mỗi bước chỉ có một token mới cần xử lý.**

Ta có sự bất đối xứng:

```text
prefill
→ đã được chuyên biệt hóa khá sâu

decode
→ vẫn chủ yếu dùng đường GEMM chung
```

Điều này đặc biệt đáng chú ý vì Q3 lại cho thấy:

> **giai đoạn sinh token chính là nơi ArcLLM chậm hơn llama.cpp rất nhiều.**

Tới đây một câu hỏi mới bắt đầu có hình dạng.

Không phải:

> “Có thể giảm vài lần giao việc cho GPU không?”

Mà là:

> **“Có phải cách ArcLLM tổ chức phép nhân ma trận cho từng token giai đoạn sinh token đang khiến GPU hoạt động rất kém hiệu quả?”**

## Vì sao mỗi lần chỉ xử lý một token có thể khó?

Hãy tưởng tượng giai đoạn xử lý đầu vào có 512 token.

GPU nhận được rất nhiều hàng dữ liệu để chia công việc:

```text
token 1
token 2
token 3
...
token 512
```

Nhưng giai đoạn sinh token thì khác.

Mỗi bước chỉ có:

```text
1 token mới
```

Nó gần với bài toán:

```text
một vector đầu vào
×
một ma trận trọng số rất lớn
```

GPU vẫn có rất nhiều đơn vị tính toán.

Nhưng giờ ta không còn hàng trăm token độc lập để chia cho chúng.

Muốn dùng GPU hiệu quả, hệ thực thi phải tìm cách chia công việc theo những hướng khác:

```text
các hàng đầu ra
chiều K của phép nhân
các block trọng số
các bước giải mã trọng số lượng tử hóa
```

Nếu cách chia không phù hợp, GPU có thể rơi vào trạng thái:

> **under-utilization — phần cứng có tài nguyên nhưng không được cung cấp công việc theo cách đủ hiệu quả để sử dụng chúng.**

Đây là lúc một thuật ngữ dài xuất hiện trong tài liệu nghiên cứu:

> **batch-1 packed-quant GEMM/dataflow under-utilization**

Ta tách nó ra:

`batch-1`

> mỗi bước chỉ xử lý một token mới.

`packed-quant`

> trọng số lượng tử hóa như Q4_K/Q6_K vẫn được giữ ở dạng đóng gói thay vì bung toàn bộ thành số thực lớn hơn.

`GEMM`

> phép nhân ma trận.

`dataflow`

> cách dữ liệu được di chuyển và tổ chức qua các bước tính toán trên phần cứng.

`under-utilization`

> GPU không được khai thác hiệu quả.

Gộp lại:

> **Cách ArcLLM hiện chia và đưa phép nhân Q4_K/Q6_K của từng token xuống GPU có thể đang khiến phần cứng hoạt động kém hiệu quả.**

Đó mới là ý nghĩa thật sự của cụm thuật ngữ dài kia.

## Nhưng một giả thuyết không được sinh ra chỉ vì nghe có lý

Sau khi Q3 đã cho kết luận âm tính, ta phải đặc biệt cẩn thận.

Nếu cứ thấy chỗ nào “có vẻ chậm” rồi viết chương trình GPU mới, ta rất dễ quay lại vòng tuning vô hạn.

Do đó việc mở SA cần hai nguồn lý do.

Thứ nhất là **bằng chứng nội tại của ArcLLM**:

```text
decode chậm hơn llama.cpp rất lớn
+
cùng model
+
cùng máy
+
decode vẫn dùng nhiều đường GEMM batch-1 dùng chung
+
prefill đã được chuyên biệt hóa sâu hơn decode
```

Thứ hai là:

> **những công trình đã được công bố công khai có cho thấy loại cơ chế này thực sự đáng nghiên cứu hay không?**

## Đọc bài báo khoa học để tìm cơ chế, không phải để mượn con số tăng tốc

Sau Q3, ta khảo sát một số công trình công khai như:

```text
FlashDecoding++
MARLIN
QServe
DeepSpeed Inference
```

Ta không cần đi sâu vào từng bài báo khoa học ở đây.

Điều quan trọng là cách dùng chúng.

Một bài báo khoa học không được phép biến thành lập luận:

```text
paper A nhanh 2×
↓
ArcLLM cũng sẽ nhanh 2×
```

Hardware khác.

chương trình GPU khác.

Driver khác.

lượng tử hóa có thể khác.

bài đo có thể khác.

Thứ duy nhất ta được mang về là:

```text
cơ chế đã được nghiên cứu
↓
nguyên lý thiết kế
↓
một câu hỏi mới cho ArcLLM
```

Không phải con số mức tăng tốc.

Ví dụ, FlashDecoding++ cho thấy các phép nhân “phẳng” trong giai đoạn sinh token có thể khai thác GPU không hiệu quả nếu dùng dataflow không phù hợp.

MARLIN cho thấy phép nhân ma trận với trọng số ít bit trong autoregressive suy luận cần scheduling, pipelining và cách giải mã trọng số được thiết kế cùng nhau.

`Pipelining — đường ống xử lý` nghĩa là:

> **chồng các giai đoạn đọc dữ liệu, giải mã và tính toán lên nhau thay vì luôn chờ bước trước hoàn tất toàn bộ mới bắt đầu bước sau.**

QServe tiếp tục củng cố một bài học:

> **trọng số dùng ít bit hơn không tự động có nghĩa hệ thực thi sẽ nhanh hơn.**

Chi phí giải mã trọng số lượng tử hóa có thể rất lớn nếu layout và cách tính không phù hợp.

`Dequantization — giải mã lượng tử hóa` là:

> **chuyển dữ liệu trọng số đã nén/lượng tử hóa sang giá trị cần thiết cho phép tính tại thời điểm chương trình GPU xử lý chúng.**

DeepSpeed suy luận lại cho thấy hiệu năng của transformer nhiều khi phải được nhìn ở mức **dataflow của cả chuỗi tính toán**, chứ không chỉ một chương trình GPU riêng lẻ.

Những công trình này không chứng minh ArcLLM sẽ nhanh hơn.

Chúng chỉ nói:

> **Giả thuyết mà ArcLLM đang đặt ra có một cơ sở kỹ thuật đủ nghiêm túc để đáng kiểm tra.**

## SA-H1 xuất hiện

Từ evidence của ArcLLM và các nghiên cứu công khai, hypothesis đầu tiên được đặt tên:

> **SA-H1 — giai đoạn sinh token-Specialized Packed-Quant Executor.**

Nói bằng tiếng Việt:

> **Một đường thực thi chuyên biệt cho giai đoạn sinh token, được thiết kế trực tiếp cho các phép nhân Q4_K/Q6_K khi mỗi bước chỉ có một token mới.**

Mục tiêu không phải đổi mô hình.

Không đổi Q4_K/Q6_K thành một loại lượng tử hóa khác.

Không bỏ qua neuron.

Không dùng cơ chế chú ý xấp xỉ.

Không đổi bài đo.

Không tìm một bài toán dễ hơn để thắng.

Câu hỏi rất hẹp:

> **Cùng mô hình, cùng phép tính, cùng ý nghĩa số học — nhưng liệu ta có thể tổ chức công việc tốt hơn cho GPU hay không?**

## Có phải chỉ cần gộp nhiều chương trình GPU lại?

Trong hệ thực thi, **chương trình GPU gộp phép tính — gộp chương trình GPU** là cách đưa hai hoặc nhiều công việc tính toán liên quan vào cùng một chương trình GPU GPU thay vì chạy chúng thành các chương trình GPU tách rời.

Ví dụ:

```text
kernel A
↓
ghi kết quả trung gian
↓
kernel B đọc lại
```

có thể, nếu semantics cho phép, được biến thành:

```text
một kernel
↓
làm A rồi tiếp tục B
```

Mục tiêu có thể là giảm số lần khởi chạy chương trình GPU, giảm dữ liệu trung gian phải ghi/đọc lại hoặc tạo điều kiện dùng chung dữ liệu đã có sẵn.

Một giả thuyết tự nhiên vì vậy là:

> “469 lần giao việc cho GPU nhiều quá. Nếu gộp nhiều phép tính liên quan lại thì có giải quyết được vấn đề không?”

SA0 kiểm tra câu này bằng một phép tính rất đơn giản.

Ba projection:

```text
Q
K
V
```

hiện cần ba lần giao việc cho GPU mỗi lớp.

Nếu gộp Q, K và V vào một chương trình GPU:

```text
3
→
1
```

mỗi lớp tiết kiệm:

```text
2 dispatch
```

28 lớp:

```text
2 × 28
= 56 dispatch
```

Gate và up cũng có thể từ:

```text
2
→
1
```

tiết kiệm thêm:

```text
1 × 28
= 28 dispatch
```

Tổng cộng:

```text
56 + 28
= 84 dispatch
```

giai đoạn sinh token đồ thị có thể từ:

```text
469
```

xuống:

```text
385
```

Nếu giả sử rất đơn giản rằng mọi lần giao việc cho GPU tốn thời gian ngang nhau, giới hạn tăng tốc theo số lượng lần giao việc cho GPU là:

```text
469 / 385
≈ 1,218×
```

Tức khoảng 22%.

Nhưng ở Q3, llama.cpp trong **phép đối chứng cùng mô hình, cùng máy và cùng bài đo** có giai đoạn sinh token thông lượng cao hơn ArcLLM khoảng:

```text
35,7× → 56,8×
```

22% không thể giải thích một khoảng cách vài chục lần.

Vì vậy SA0 loại bỏ một lời giải thích quá đơn giản:

> **Chỉ giảm số lần giao việc cho GPU bằng cách gộp chương trình GPU không thể là cơ chế chính.**

Gộp chương trình GPU vẫn có thể hữu ích.

Nhưng nó chỉ là yếu tố hỗ trợ.

Cơ chế chính vẫn phải nằm sâu hơn ở:

> **cách tổ chức phép nhân ma trận, đọc trọng số, giải mã trọng số và chia công việc trên GPU.**

## Tiết kiệm việc đọc dữ liệu trung gian cũng chưa đủ

Q, K và V cùng đọc một hidden vector.

Hidden width là:

```text
3584 giá trị F32
```

`F32` là số thực 32-bit, mỗi giá trị chiếm:

```text
4 byte
```

Một vector:

```text
3584 × 4
= 14.336 byte
```

Nếu việc gộp QKV giúp tránh được hai lần đọc thừa mỗi lớp:

```text
2 × 14.336 × 28
= 802.816 byte/token
```

Việc gộp gate/up tránh thêm:

```text
14.336 × 28
= 401.408 byte/token
```

Tổng cộng:

```text
1.204.224 byte/token
≈ 1,2 MB/token
```

Nghe có vẻ lớn.

Nhưng phần trọng số mà SA0 dùng làm proxy đã hơn:

```text
4,37 GB
```

1,2 MB chỉ khoảng:

```text
0,0276%
```

của quy mô đó.

Do vậy cũng không thể nói:

> “Gộp chương trình GPU sẽ giải quyết hiệu năng vì nó giảm rất nhiều lượng dữ liệu mô hình phải đọc.”

Bằng chứng không hỗ trợ kết luận ấy.

Việc gộp chương trình GPU có thể giúp vì những lý do khác:

```text
giảm dữ liệu trung gian
giảm một số bước chuẩn bị lặp lại
tạo điều kiện chia việc tốt hơn
tạo điều kiện xây pipeline giải mã trọng số tốt hơn
```

Nhưng cơ chế chính vẫn phải là cách tổ chức phép nhân Q4_K/Q6_K cho từng token.

## Chính llama.cpp cho ta một bằng chứng rất quan trọng

Q3 có một lợi thế đặc biệt.

Nó không chỉ nói ArcLLM chậm.

Nó còn có một hệ thực thi đối chứng chạy trên **chính máy đó**.

ArcLLM giai đoạn sinh token:

```text
≈ 0,297 → 0,335 token/giây
```

llama.cpp trên cùng mô hình, cùng Intel Arc 140V và cùng bài đo:

```text
≈ 11,35 → 18,58 token/giây
```

Khoảng cách:

```text
≈ 35,7× → 56,8×
```

Những con số này không cho phép kết luận:

> “GEMM chính là nguyên nhân của toàn bộ khoảng cách.”

Nhưng chúng cho phép một kết luận quan trọng hơn:

> **Intel Arc 140V không bị giới hạn ở mức tốc độ mà ArcLLM hiện đang đạt.**

Chính phép đối chứng với llama.cpp đã chứng minh rằng:

> **cùng mô hình và cùng phần cứng có thể sinh token nhanh hơn rất nhiều so với đường giai đoạn sinh token hiện tại của ArcLLM.**

Vấn đề vì vậy nằm ở cách hệ thực thi hiện tổ chức công việc, chứ không thể đơn giản đổ cho giới hạn tuyệt đối của GPU.

## SA-H1 được tách nhỏ trước khi triển khai

SA0 tiếp tục chia giả thuyết lớn thành những câu hỏi nhỏ hơn.

Cơ chế chính được giữ lại là:

> **GPU có thể đang bị khai thác kém hiệu quả vì cách ArcLLM tổ chức GEMM Q4_K/Q6_K cho từng token giai đoạn sinh token.**

Đây là phần quan trọng nhất của SA-H1.

Việc gộp QKV và gộp gate/up được giữ như những cơ chế hỗ trợ.

Chúng có thể giúp:

```text
giảm dữ liệu trung gian
giảm một số bước chuẩn bị lặp lại
tạo điều kiện chia việc tốt hơn
tạo điều kiện xây pipeline giải mã trọng số tốt hơn
```

nhưng không được coi là lời giải chính.

LM head cũng chưa được chọn làm mục tiêu đầu tiên.

Lý do rất đơn giản:

> **Chưa có bằng chứng theo từng giai đoạn cho thấy LM head là nơi đáng tấn công trước.**

Không tối ưu chỉ vì một phần nhìn có vẻ lớn.

## Và vẫn chưa được phép viết chương trình GPU mới

Đến đây ta đã có:

```text
Q3 đóng kiến trúc cũ
↓
xác định vùng decode đáng nghi
↓
đối chiếu với nghiên cứu đã công bố
↓
hình thành SA-H1
↓
phân rã cơ chế
```

Nhưng chương trình GPU mới vẫn chưa được phép xuất hiện.

Đó là vai trò của:

> **SA0 — bước kiểm tra nguyên nhân và khả năng thực hiện trước khi đầu tư vào triển khai.**

SA0 không xác nhận rằng kiến trúc mới nhanh hơn.

Nó chỉ hỏi:

> **Giả thuyết này có đủ cơ sở để đáng tiêu thêm thời gian nghiên cứu hay không?**

Đây là một loại `PASS` khác hoàn toàn với những chương đầu.

Ở Chương 4, một phép tính:

```text
tính đúng
→ PASS
```

Ở SA0:

```text
cơ chế có lý
+
phần cứng có khả năng hỗ trợ
+
có cách kiểm tra rõ ràng
→ PASS để tiếp tục nghiên cứu
```

Nó **không phải ĐẠT (PASS) hiệu suất**.

Nó cũng không có nghĩa giả thuyết đã đúng.

Chỉ có nghĩa:

> **chưa có lý do đủ mạnh để loại giả thuyết trước khi làm thí nghiệm thành phần.**

## Phần cứng thật phải tự trả lời

SA-H1 có thể cần những khả năng phần cứng như:

```text
subgroup
FP16
INT8
cooperative matrix
```

Nhưng ta không được nhìn tên:

```text
Intel Arc 140V
```

rồi tự giả định những thứ đó tồn tại.

Vì vậy SA0 có một bước riêng:

> **SA0-CAP — kiểm tra khả năng phần cứng của SA0.**

`CAP` ở đây là viết tắt của **capability — khả năng phần cứng có thể cung cấp**.

Trong các tài liệu GPU/API, từ **primitive — thao tác nền tảng** thường được dùng cho những khả năng cơ bản mà phần cứng hoặc API cung cấp để các phép tính lớn hơn xây lên trên đó. Ví dụ một loại thao tác theo nhóm lane, một kiểu dữ liệu số học hay một phép toán ma trận chuyên biệt đều có thể được xem là primitive ở mức này.

SA0-CAP không chạy mô hình.

Không phép đo so sánh.

Không tạo shader mới cho successor.

Nó chỉ hỏi đúng GPU và đúng driver:

> **“Trên máy này, phần cứng thực sự hỗ trợ những thao tác nền tảng nào?”**

Kết quả trên target machine xác nhận những khả năng nền tảng như:

```text
Vulkan compute
→ có

subgroup
→ có

subgroup size
→ 32

shared memory cho compute workgroup
→ 49.152 byte

tối đa workgroup invocation
→ 1024
```

Ngoài ra device còn công bố hỗ trợ:

```text
FP16
INT8
subgroup-size control
cooperative matrix
```

`Subgroup` có thể hiểu gần đúng là:

> **một nhóm nhỏ các lane GPU có thể phối hợp chặt chẽ khi thực hiện cùng một công việc.**

`Cooperative matrix` là:

> **khả năng phần cứng hỗ trợ một số dạng phép toán ma trận theo nhóm một cách chuyên biệt.**

Nhưng cần giữ ranh giới:

> **GPU hỗ trợ cooperative matrix không có nghĩa cooperative matrix chắc chắn là cách tốt nhất để chạy Q4_K/Q6_K.**

Khả năng phần cứng chỉ nói:

> **con đường tồn tại.**

Nó chưa nói:

> **con đường nhanh.**

## SA0 cuối cùng đã chứng minh được gì?

Không phải:

> “Kiến trúc kế tiếp nhanh hơn.”

Không phải:

> “SA-H1 đúng.”

Không phải:

> “ArcLLM sắp thắng llama.cpp.”

Điều SA0 thực sự cho phép nói chỉ là:

```text
cơ sở nhân quả
→ đủ hợp lý

khả năng phần cứng
→ có

gộp kernel đơn thuần
→ không đủ làm lời giải chính

đường GEMM Q4_K/Q6_K batch-1 chuyên biệt
→ đáng tiếp tục kiểm tra

hiệu suất của kiến trúc kế tiếp
→ chưa biết
```

Đây là một kết quả rất quan trọng dù chưa có mức tăng tốc nào.

Bởi câu hỏi ban đầu:

> “Làm ArcLLM nhanh hơn bằng cách nào?”

đã được thu hẹp thành:

> **“Một cách tổ chức khác cho phép nhân Q4_K/Q6_K của từng token có thể khai thác GPU hiệu quả hơn mà vẫn giữ đúng kết quả hay không?”**

Câu hỏi mới nhỏ hơn.

Có phạm vi.

Có điều kiện dừng.

Có khả năng KHÔNG ĐẠT (FAIL).

Và quan trọng nhất:

> **nó không còn là lời mời cho AI thử vô hạn mọi ý tưởng có thể nghĩ ra.**

## Phần II kết thúc ở đây

Hãy nhìn lại ba chương vừa qua.

Chương 9 hỏi:

> **Ta đang đứng ở đâu khi so với một hệ thực thi trưởng thành dưới cùng điều kiện?**

Chương 10 hỏi:

> **Khoảng cách đó có còn chứa một lợi thế thực tế nào đủ lớn và tái lập hay không?**

Bằng chứng trả lời:

> **Không chứng minh được.**

Kiến trúc hiện tại phải đóng.

Chương 11 không cố đảo kết quả ấy.

Nó hỏi một câu hoàn toàn mới:

> **Có đủ cơ sở để mở một giả thuyết kiến trúc kế tiếp hay không?**

Và SA0 trả lời:

> **Có — nhưng chỉ đủ để tiếp tục kiểm tra. Chưa đủ để tuyên bố thành công.**

Toàn bộ Phần II có thể thu lại thành:

```text
runtime chạy được
↓
đưa vào phép đối chứng cùng điều kiện
↓
đo khoảng cách
↓
khóa tiêu chuẩn trước bằng chứng mới
↓
không chứng minh được lợi thế
↓
đóng kiến trúc hiện tại
↓
chỉ mở câu hỏi mới khi có cơ chế đủ rõ
```

Đây là điểm rất khác giữa:

> **xây một hệ thống**

và:

> **nghiên cứu một hệ thống.**

Xây hệ thống thường hỏi:

> “Làm sao để nó chạy?”

Nghiên cứu phải hỏi thêm:

> “Bằng chứng nào sẽ khiến ta dừng?”

Và khi câu trả lời xuất hiện, ta phải thực sự dừng.

### Nhớ 3 điều

1. **Successor Architecture (SA) không phải một “v2” mặc định.** Kiến trúc kế tiếp chỉ được mở khi kiến trúc cũ đã đóng và có một cơ chế mới đủ cụ thể để kiểm tra.
2. **Các công trình đã công bố chỉ giúp hình thành giả thuyết.** mức tăng tốc của FlashDecoding++, MARLIN, QServe hay DeepSpeed không phải bằng chứng rằng ArcLLM sẽ đạt cùng kết quả trên Intel Arc.
3. **SA0 chưa chứng minh hiệu suất.** Nó chỉ xác nhận rằng giả thuyết về cách tổ chức GEMM Q4_K/Q6_K cho từng token có đủ cơ sở về cơ chế và phần cứng để đáng tiếp tục nghiên cứu.

> **Phần II kết thúc tại đây.**
>
> Ta đã xây được một hệ thực thi, để bằng chứng phán xét nó, chấp nhận một kết quả âm tính và học được cách đặt một câu hỏi mới mà không phủ nhận kết quả cũ.
