# Chương 18 — Dữ liệu ở trong bộ nhớ vẫn chưa đủ: lấy từ đâu và sống bao lâu

> **Mức đọc: Nâng cao**
>
> **Bản đồ xuyên suốt**
>
> ```text
> HỌ HÀNG KHÁI NIỆM                    ĐƯỜNG ĐI CỦA TOKEN / RUNTIME
> 
> AI                                   Văn bản
> ↓                                    ↓
> Machine Learning                     Tokenizer
> ↓                                    ↓
> Neural Network                       Token / token ID
> ↓                                    ↓
> Language Model                       Embedding → tensor
> ↓                                           +
> LLM                                  parameters / weights từ model
> ↓                                           ↓
> Transformer                          Runtime
> ↓                                           ↓
> Decoder-only Transformer             CPU / GPU / bộ nhớ
> ↓                                           ↓
> Nhiều decoder layer                  RMSNorm / Attention / FFN
> ↓ chứa                                      ↓
> Parameters / Weights                 một decoder layer
>                                             ↓
>                                      nhiều decoder layer
>                                             ↓
>                                      logits → token tiếp theo
>                                             ↓
>                                      KV cache / lặp lại
>                                             ↓
>                                      benchmark / tối ưu
>                                             ↓
>                                      representation / lifecycle
> ```
>
> ▶ **Đang mở ở chương này:** residency / acquisition / readiness / lifecycle.


> **Câu hỏi của chương:** Nếu runtime biết một tensor hoặc một cách biểu diễn dữ liệu đang tồn tại trong bộ nhớ, thông tin đó đã đủ để quyết định có thể dùng nó ngay cho phép tính hay chưa?

Ở Chương 6, một bước tiến rất lớn của ArcLLM là giữ trọng số mô hình trong vùng bộ nhớ mà GPU có thể truy cập.

Khi ấy, câu hỏi chủ yếu là:

> **Dữ liệu đã ở đó chưa?**

Chương 17 làm câu hỏi này phức tạp hơn.

Cùng một tensor logic giờ có thể có:

```text
Q4_K gốc
```

và:

```text
EXEC148
```

Cả hai mang cùng nội dung trọng số.

Nhưng một đường thực thi có thể dùng Q4_K gốc, còn đường khác lại cần EXEC148.

Vì vậy chỉ biết:

```text
tensor đang ở trong bộ nhớ
```

không còn đủ.

Runtime phải biết thêm:

```text
cách biểu diễn nào đang tồn tại?

cách nào phép tính hiện tại cần?

nếu chưa có thì có tạo được không?

tạo bằng cách nào?

sau khi tạo có dùng được ngay không?

giữ nó tới bao giờ?

khi nào phải bỏ?
```

Đây là lúc ba khái niệm bắt đầu tách ra rõ ràng:

> **trạng thái cư trú trong bộ nhớ (residency)** — một cách biểu diễn đang có mặt trong vùng bộ nhớ cần thiết hay chưa.

> **quá trình thu nhận hoặc tạo biểu diễn (acquisition)** — làm cho cách biểu diễn cần thiết xuất hiện.

> **vòng đời (lifecycle)** — khi nào giữ, khi nào loại bỏ và khi nào phải tạo lại cách biểu diễn đó.

Nhưng bằng chứng sau đó còn cho thấy ngay cả ba khái niệm này vẫn chưa đủ nếu ta trộn chúng với câu hỏi:

> **Phép tính có thực sự chạy được ngay lúc này không?**

## “Có trong bộ nhớ” và “dùng được ngay” không phải một

Hãy lấy EXEC148.

Giả sử runtime biết:

```text
EXEC148 đã tồn tại trong bộ nhớ
```

Điều đó nói rằng vùng dữ liệu cần thiết đang có mặt.

Nhưng ta vẫn có thể gặp tình huống:

- đường thực thi tương ứng chưa khả dụng;
- tài nguyên cần thiết tạm thời chưa sẵn sàng;
- cách biểu diễn đã hết hiệu lực;
- mô hình hiện tại không khớp với cách biểu diễn;
- hoặc yêu cầu nằm ngoài phạm vi đã được kiểm chứng.

Vì vậy:

```text
đang cư trú trong bộ nhớ
```

không đồng nghĩa:

```text
sẵn sàng thực thi
```

Nói ngắn gọn:

> **Có dữ liệu trong bộ nhớ không có nghĩa phép tính đã sẵn sàng chạy.**

Chiều ngược lại cũng đúng.

Một phép tính có thể chạy được mà **không cần một cách biểu diễn phụ nào cả**.

Điều đó sau này trở thành một phản ví dụ rất quan trọng.

Nhưng trước hết, hãy xem việc tạo biểu diễn thực sự có nhiều loại hơn ta tưởng.

## Có biểu diễn phụ là tùy chọn trong một trường hợp

Với Q4-down ở Chương 16 và 17, runtime có hai con đường hợp lệ.

Con đường A:

```text
Q4_K gốc
↓
Split-K32
```

Không cần EXEC148.

Con đường B:

```text
Q4_K gốc
↓
tạo EXEC148
↓
Serial-K đọc EXEC148
```

B nhanh hơn trong phạm vi đã đo.

Nhưng B phải trả khoảng:

```text
231,64 ms
```

để tạo biểu diễn một lần, cùng khoảng:

```text
550 MB
```

vùng dữ liệu bổ sung trong thí nghiệm.

Vì vậy nếu EXEC148 chưa tồn tại, runtime không nhất thiết phải tạo nó.

Nó có thể hỏi:

> **Ta còn đủ nhiều token để lợi ích tương lai bù được chi phí tạo hay không?**

Nếu câu trả lời là không, dùng A.

Nếu câu trả lời là có, tạo B rồi dùng B.

Đây là loại thứ nhất:

> **tạo biểu diễn dựa trên khả năng bù chi phí nhờ tái sử dụng (reuse-amortized acquisition).**

Nói đơn giản:

> **Chỉ tạo một biểu diễn tốn kém nếu dự kiến dùng nó đủ lâu để thu hồi chi phí ban đầu.**

Ta đã thấy ví dụ ở Chương 17.

Với W-S, nếu chỉ còn:

```text
10 token
```

thì A còn rẻ hơn.

Khoảng sau:

```text
16,3 token
```

B mới bắt đầu bù được chi phí tạo trong mô hình thời gian đơn giản đó.

Vì vậy EXEC148 ở trường hợp này là:

> **một cách biểu diễn hữu ích nhưng không bắt buộc.**

Nếu không có nó, runtime vẫn còn A.

## Nhưng có biểu diễn phụ lại là bắt buộc trong trường hợp khác

Sau đó ArcLLM gặp một họ bài toán khác trong nhánh P8.

Điểm quan trọng là obstruction ở đây **không phải tổng dung lượng bộ nhớ không đủ**.

Với exact model 7B, P8-A tính được:

```text
tổng residency dự kiến
= 5.347.770.372 byte

usable budget đã khóa
= 16.374.562.816 byte

headroom
= 11.026.792.444 byte
```

Tức **capacity tổng thể PASS**.

FAIL nằm ở một contract hẹp hơn đã được kế thừa từ kiến trúc trước:

> **mỗi physical arena / tensor piece không được vượt 256 MiB.**

Hai tensor vocab đơn lẻ vi phạm contract đó:

```text
token_embd.weight

output.weight
```

P8-A2 không nới arena cap, không đổi quantization, context hay KV precision để cứu kết quả.

Nó thay cách **biểu diễn vật lý** của đúng hai logical tensor lớn đó:

```text
một logical tensor lớn
↓
nhiều physical segment
↓
chỉ cắt tại ranh giới hàng
↓
mỗi segment <= 256 MiB
```

Đây là **row-aligned physical segmentation — phân đoạn vật lý theo ranh giới hàng**.

Tổng dữ liệu logic không đổi.

Tổng công thức bộ nhớ không được cứu bằng cách làm nhỏ model.

Chỉ cách cùng tensor logic được ánh xạ thành các physical piece thay đổi để contract arena vẫn được giữ.

Trong nghiên cứu Phase2 về semantics của runtime, chính trường hợp P8 có giới hạn này được dùng như một:

> **bounded mandatory-feasibility oracle — một trường hợp đối chứng có phạm vi giới hạn, trong đó representation cần thiết là điều kiện để đường thực thi đó khả thi.**

Không có một đường dự phòng đã được xác nhận tương đương như A trong trường hợp EXEC148.

Hình ảnh gần với:

```text
representation bắt buộc chưa có
↓
nếu có acquisition hợp lệ
→ tạo / thu nhận representation

nếu hiện không thể acquisition
→ NOT_READY
```

Trong trường hợp này, câu hỏi:

> “Có đáng tạo biểu diễn nếu sẽ dùng nhiều token không?”

là câu hỏi sai.

Bởi việc tạo representation không phải một tối ưu tùy chọn để hoàn vốn.

Nó là điều kiện để đường thực thi bounded đó trở nên khả thi.

Đây là loại thứ hai:

> **tạo biểu diễn bắt buộc để phép tính trở nên khả thi (mandatory-for-feasibility acquisition).**

Ta có thể so hai trường hợp:

```text
TRƯỜNG HỢP 1

EXEC148 chưa có
↓
vẫn có A
↓
có thể chọn:
dùng A
hoặc tạo B nếu đáng
```

với:

```text
TRƯỜNG HỢP 2

representation bắt buộc chưa có
↓
không có đường dự phòng đã xác nhận
↓
phải acquisition
hoặc NOT_READY
```

Hai loại này hoàn toàn khác nhau.

Cũng phải giữ đúng biên giới claim:

> **Phase2 dùng P8 như một bounded oracle cho semantics acquisition; điều đó không tự nó biến P8 thành một claim full-inference mới.**

## Không được ép mọi cách tạo biểu diễn vào một công thức

Một tư duy dễ xuất hiện là:

> “Mọi biểu diễn đều có chi phí tạo, vậy cứ dùng một ngưỡng tái sử dụng cho tất cả.”

Nhưng trường hợp bắt buộc để khả thi đã bác bỏ điều đó.

Giả sử một cách biểu diễn là điều kiện để phép tính tồn tại.

Dù tương lai chỉ có:

```text
1 token
```

hay thậm chí chưa biết sẽ có bao nhiêu token, runtime vẫn phải thử tạo nó nếu muốn đi con đường đó.

Không thể viết:

```text
tái sử dụng thấp
→ không tạo
```

cho mọi trường hợp.

Vì vậy mô hình phải tách:

```text
LOẠI 1
tùy chọn
↓
có thể dùng ngưỡng tái sử dụng

LOẠI 2
bắt buộc để khả thi
↓
không được gắn ngưỡng tái sử dụng
```

Đây là một ví dụ điển hình của cách một lớp trừu tượng trưởng thành.

Không phải ta ngồi nghĩ từ đầu:

> “Có lẽ nên có hai loại.”

Mà là một họ thực tế thứ hai cho thấy:

> **Một loại duy nhất không còn diễn tả đúng bằng chứng.**

## Nếu hiện tại không thể tạo thì sao?

Trường hợp bắt buộc đặt ra một trạng thái mới.

Nếu biểu diễn chưa có nhưng có thể tạo:

```text
TẠO / THU NHẬN BIỂU DIỄN
```

Nếu biểu diễn chưa có và hiện không thể tạo:

```text
NOT_READY
```

Nói bằng tiếng Việt:

> **Chưa sẵn sàng để chạy.**

Điều quan trọng là runtime không được tự chế một đường dự phòng không có trong bằng chứng.

Ví dụ:

```text
biểu diễn bắt buộc chưa có
+
không thể tạo lúc này
```

không được biến thành:

```text
chạy bằng một đường khác
chưa từng được xác nhận
```

Kết quả đúng có thể đơn giản là:

> **Hiện tại không có đường thực thi hợp lệ.**

Đây là một bước trưởng thành quan trọng.

Runtime không chỉ cần biết:

> “Đường nào nhanh nhất?”

Nó còn phải có khả năng trả lời:

> **“Không có đường hợp lệ để chạy ngay lúc này.”**

## Còn một trạng thái khác: nằm ngoài bằng chứng

Có một lý do khác khiến runtime không được đưa yêu cầu vào một đường thực thi.

Giả sử khối chức năng đó chỉ được xác nhận với:

```text
Q4_K
một hình dạng tensor cụ thể
một mô hình cụ thể
một phạm vi công việc cụ thể
```

Nhưng yêu cầu mới nằm ngoài phạm vi đó.

Về mặt kỹ thuật, đường thực thi có thể vẫn chạy.

Nhưng nghiên cứu chưa chứng minh rằng ta được phép coi kết quả đó là hợp lệ.

Khi ấy trạng thái đúng là:

> **OUTSIDE_VALIDATED_CAPABILITY — nằm ngoài phạm vi khả năng đã được xác nhận.**

Nói đơn giản:

```text
không phải
“không chạy được”

mà là
“ta chưa có bằng chứng để dùng đường này cho trường hợp đó”
```

Hai trạng thái phải được tách:

```text
NOT_READY
→ nằm trong phạm vi hợp lệ
  nhưng hiện chưa thể chạy

OUTSIDE_VALIDATED_CAPABILITY
→ yêu cầu nằm ngoài phạm vi đã được chứng minh
```

Đây là cách runtime bắt đầu mang ranh giới của bằng chứng khoa học vào chính quyết định thực thi.

## Tới đây tưởng đã đủ — nhưng cơ chế trực tiếp ở Chương 14 phá tiếp mô hình

Sau khi tách hai loại tạo biểu diễn, mô hình đã xử lý được:

1. cách biểu diễn tùy chọn như EXEC148;
2. cách biểu diễn bắt buộc để phép tính khả thi.

Nhưng rồi ArcLLM lấy chính cơ chế đã được xác nhận ở Chương 14 làm một phép thử độc lập.

Nhắc lại:

```text
Q4_K gate/up
↓
Split-K32
↓
56 node decode
```

Cơ chế này thay **cách thực thi**.

Nó không tạo một biểu diễn mới.

Không có EXEC148.

Không có vùng dữ liệu phụ.

Không cần bước tạo biểu diễn.

Không có vòng đời của một representation mới.

Nếu đường Split-K32 tồn tại và dùng được:

> **chạy nó ngay.**

Nếu nó không khả dụng:

> **dùng đường đối chứng dự phòng đã được xác nhận.**

Đây là một trường hợp rất đơn giản.

Nhưng chính sự đơn giản đó làm mô hình cũ FAIL.

## Sai lầm: dùng trạng thái cư trú để trả lời “có chạy được không?”

Mô hình lúc đó vẫn còn một giả định ngầm:

> Muốn đường thực thi ưu tiên được chọn thì representation của nó phải đang cư trú trong bộ nhớ.

Điều này hợp lý với EXEC148.

Bởi B thật sự cần một vùng dữ liệu riêng.

Nó cũng có thể diễn tả họ biểu diễn bắt buộc.

Nhưng với cơ chế Split-K32 của Chương 14:

```text
không có representation phụ
```

nên câu hỏi:

```text
representation có đang resident không?
```

thậm chí không có ý nghĩa.

Nếu vẫn bắt cơ chế này đi qua cùng logic, chỉ còn ba cách “lách”.

### Cách 1

Giả vờ:

```text
resident = true
```

dù chẳng có representation phụ nào đang tồn tại.

Như vậy từ `resident` đã bị đổi nghĩa.

### Cách 2

Bịa ra một bước tạo dữ liệu.

Nhưng cơ chế này không tạo thêm dữ liệu nào cả.

### Cách 3

Biến đường ưu tiên thành chính đường dự phòng của nó.

Nhưng như vậy mất luôn đường đối chứng dự phòng thật.

Cả ba đều sai.

Không phải code sai.

Mà lớp trừu tượng sai.

## Một FAIL rất có giá trị

Kết quả của phép thử độc lập này là:

> **Mô hình chưa tổng quát đủ.**

Không phải cơ chế Split-K32 sai.

Không phải khái niệm tạo biểu diễn sai.

Mà là:

> **Runtime đang dùng trạng thái “representation có trong bộ nhớ” để trả lời một câu hỏi rộng hơn khả năng của nó: “đường thực thi có thể chạy ngay không?”**

Đây là hai câu hỏi khác nhau.

Bằng chứng buộc ArcLLM phải tách chúng.

## Một trạng thái mới: sẵn sàng thực thi

Khái niệm được thêm vào rất nhỏ:

> **trạng thái sẵn sàng thực thi (execution readiness)** — đường thực thi có thể xử lý yêu cầu hiện tại ngay bây giờ hay không.

Nó là một trạng thái độc lập với việc có hay không có representation phụ trong bộ nhớ.

Từ đây có bốn câu hỏi tách biệt:

```text
1. đường thực thi có tồn tại không?

2. đường đó có sẵn sàng chạy yêu cầu này ngay không?

3. representation phụ có đang ở trong bộ nhớ không?

4. nếu chưa có, có con đường nào để tạo hoặc thu nhận nó không?
```

Ta cũng cần phân biệt:

> **khả dụng về nguyên tắc (execution available)** — đường thực thi tồn tại và về nguyên tắc có thể dùng.

với:

> **sẵn sàng thực thi (execution ready)** — đường đó có thể xử lý yêu cầu hiện tại ngay lúc này.

Một đường có thể tồn tại trong runtime nhưng tạm thời chưa sẵn sàng cho yêu cầu hiện tại.

## Ba ví dụ làm mọi thứ rõ hơn

### Ví dụ 1 — cơ chế trực tiếp, không cần representation phụ

```text
đường Split-K32
→ tồn tại

sẵn sàng thực thi
→ có

representation phụ
→ không có và không cần

bước tạo biểu diễn
→ không có
```

Kết quả:

```text
chạy ngay bằng Split-K32
```

Nếu Split-K32 không sẵn sàng:

```text
chuyển sang đường dự phòng
```

Trạng thái cư trú trong bộ nhớ không tham gia.

### Ví dụ 2 — EXEC148 chưa tồn tại nhưng A còn dùng được

```text
đường B
→ tồn tại

EXEC148
→ chưa có trong bộ nhớ

có thể tạo B
→ có

đường dự phòng A
→ hợp lệ
```

Runtime phải cân nhắc chính sách:

```text
tái sử dụng thấp
→ dùng A

tái sử dụng đủ cao
→ tạo B
→ xác minh
→ dùng B
```

Ở đây trạng thái cư trú và quá trình tạo biểu diễn đều quan trọng.

### Ví dụ 3 — biểu diễn bắt buộc chưa có

```text
đường thực thi
→ nằm trong phạm vi hợp lệ

biểu diễn bắt buộc
→ chưa có trong bộ nhớ

có thể tạo
→ có
```

Kết quả:

```text
TẠO BIỂU DIỄN
```

Không hỏi ngưỡng tái sử dụng.

Nếu hiện không thể tạo:

```text
NOT_READY
```

Không được bịa ra đường dự phòng.

Ba trường hợp này cho thấy:

> **Không một biến “đang ở trong bộ nhớ” nào có thể mô tả đúng tất cả.**

## Trạng thái cư trú chỉ nên có một nghĩa

Sau nhiều lần bị phản ví dụ làm hỏng mô hình, một nguyên tắc tốt xuất hiện:

> **Một trạng thái chỉ nên trả lời đúng một câu hỏi.**

`Resident` từ đây chỉ có nghĩa:

> **Một representation được tạo riêng có đang tồn tại trong vùng bộ nhớ cần thiết hay không.**

Không dùng trạng thái này để nói:

- đường thực thi tồn tại;
- đường thực thi chạy được;
- khối chức năng nằm trong phạm vi hợp lệ;
- bước tạo representation có thể thực hiện;
- hay yêu cầu nằm trong phạm vi bằng chứng.

Từ đây ta có thể nhìn từng chiều riêng:

```text
DANH TÍNH
đây là khối chức năng / cách biểu diễn nào?

KHẢ DỤNG THỰC THI
đường thực thi có tồn tại không?

SẴN SÀNG THỰC THI
có chạy được yêu cầu hiện tại ngay không?

TRẠNG THÁI CƯ TRÚ
representation phụ có đang ở trong bộ nhớ không?

QUÁ TRÌNH THU NHẬN
nếu thiếu, có thể lấy hoặc tạo bằng cách nào?

VÒNG ĐỜI
sau đó giữ hay bỏ khi nào?
```

Chương 19 sẽ gom sáu chiều này thành một bề mặt runtime tổng quát hơn.

Nhưng trước đó còn một phần quan trọng:

> **Dữ liệu đã được tạo rồi thì sống bao lâu?**

## Tạo xong mới chỉ là lúc bắt đầu của vòng đời

Giả sử EXEC148 đã được tạo.

Runtime xác minh nó đúng với mô hình.

Đường B dùng được.

Sau một yêu cầu, có nên giải phóng ngay không?

Không có đáp án chung.

Nếu yêu cầu tiếp theo sắp tới, giữ lại có thể tránh:

```text
231,64 ms
```

tạo lại.

Nhưng giữ lại tốn gần:

```text
550 MB
```

trong thí nghiệm hiện tại.

Nếu **áp lực bộ nhớ (memory pressure)** tăng, giữ representation này có thể làm phần khác của hệ thống khó hoạt động.

Vì vậy vòng đời phải là một quyết định riêng.

Có thể có các kiểu:

```text
tạo khi nạp mô hình
```

hoặc:

```text
tạo khi dùng lần đầu
```

hoặc:

```text
giữ trong một phiên làm việc
```

hoặc:

```text
giữ qua nhiều phiên
nếu còn hợp lệ và bộ nhớ cho phép
```

Điều quan trọng là:

> **Cách representation được tạo không quyết định duy nhất nó phải sống bao lâu.**

CPU có thể tạo nhưng representation được giữ qua nhiều phiên.

GPU có thể tạo nhưng representation chỉ sống trong một yêu cầu.

Hai chiều đó độc lập.

## Khi nào phải bỏ?

Một vòng đời hợp lý phải có điều kiện loại bỏ.

Ví dụ:

```text
dỡ mô hình khỏi bộ nhớ
→ representation đi kèm không còn cần thiết
```

Hoặc:

```text
representation không còn đúng danh tính
→ bỏ
```

Hoặc:

```text
áp lực bộ nhớ vượt chính sách
→ có thể loại representation tùy chọn
```

Thao tác loại representation khỏi bộ nhớ thường được gọi là:

> **evict — loại khỏi bộ nhớ**.

Nếu sau này cần lại, runtime có thể phải tạo hoặc nạp lại.

Nhưng cũng phải cẩn thận:

> **Một đường thực thi tạm thời chưa sẵn sàng không mặc nhiên có nghĩa representation phải bị xóa.**

Giả sử EXEC148 vẫn hoàn toàn hợp lệ trong bộ nhớ, nhưng GPU tạm thời chưa thể dùng đường B.

Nếu xóa EXEC148 ngay chỉ vì trạng thái sẵn sàng thực thi chuyển thành “không”, ta có thể phá mất một vùng dữ liệu đắt tiền mà lát nữa có thể dùng lại.

Vì vậy:

```text
tạm thời chưa sẵn sàng thực thi
≠
tự động loại representation khỏi bộ nhớ
```

Trạng thái thực thi và vòng đời dữ liệu phải tiếp tục tách nhau.

## Một ví dụ nhỏ về giá trị của việc giữ lại

Giả sử tạo EXEC148 mất:

```text
232 ms
```

Phiên đầu dùng đủ lâu để B có lợi.

Nếu cuối phiên ta xóa ngay representation, phiên kế tiếp lại trả:

```text
232 ms
```

Nếu có 5 phiên liên tiếp:

```text
5 × 232
=
1.160 ms
```

chỉ riêng chi phí tạo lại.

Nếu representation vẫn hợp lệ và bộ nhớ không căng, giữ nó qua các phiên có thể tránh phần chi phí này.

Nhưng nếu 550 MB đó làm hệ thống thiếu bộ nhớ, lựa chọn lại có thể đảo chiều.

Đây chính là lý do vòng đời không thể chỉ là:

```text
tạo
↓
dùng
↓
xóa
```

Runtime cần một **chính sách vòng đời**.

## Lớp kết nối phần cứng thật có tuân theo các ranh giới này không?

Một lớp trừu tượng trên giấy vẫn chưa đủ.

Sau khi các khái niệm được tách, ArcLLM còn phải hỏi:

> **Một lớp kết nối Vulkan thật có tuân theo những ranh giới đó được không?**

`Backend` trong ngữ cảnh này có thể hiểu là:

> **lớp kết nối các quyết định chung của runtime với cơ chế thực thi cụ thể trên phần cứng.**

Khi gắn mô hình vào đường Q4 thật, các trường hợp sau đã được kiểm tra:

```text
A không cần tạo representation phụ
→ PASS
```

```text
đường B chưa khả dụng
→ không âm thầm thử lại bằng một cơ chế ẩn
→ PASS
```

```text
chính sách chọn tạo B
→ tạo
→ kiểm tra danh tính
→ dùng B
→ PASS
```

```text
B đã ở trong bộ nhớ và còn hợp lệ
→ dùng lại
→ không tạo lần nữa
→ PASS
```

```text
B bị loại khỏi bộ nhớ
→ quay về A khi phù hợp
→ PASS
```

Điểm quan trọng không nằm ở số lần cấp phát hay giải phóng bộ nhớ.

Nó nằm ở việc:

> **Lớp kết nối phần cứng thật có thể tuân thủ đúng sự phân biệt giữa chọn đường thực thi, tạo representation, trạng thái sẵn sàng, trạng thái cư trú và vòng đời mà không cần lén thêm luật riêng cho Q4.**

Đó là dấu hiệu lớp trừu tượng bắt đầu có giá trị kiến trúc thật.

## Không phải mọi trạng thái đều cần một “máy trạng thái” khổng lồ

Khi thấy nhiều khái niệm như vậy, phản xạ thiết kế có thể là tạo một danh sách trạng thái lớn:

```text
ABSENT
ACQUIRING
RESIDENT
READY
BLOCKED
STALE
EVICTED
...
```

Nhưng ArcLLM không đi theo hướng đó chỉ vì có thể.

Bằng chứng tại thời điểm này cho thấy chỉ cần thêm một giá trị đúng/sai độc lập:

```text
execution_ready
```

là đủ để xử lý ba họ đã được xác nhận:

1. representation tùy chọn có đường dự phòng;
2. representation bắt buộc để khả thi;
3. cơ chế thực thi trực tiếp không cần representation phụ.

Không có bằng chứng để biện minh cho một mô hình trạng thái lớn hơn.

Đây lại là nguyên tắc:

> **Chỉ thêm độ phức tạp mà phản ví dụ thực tế buộc ta phải thêm.**

## Một lớp trừu tượng trưởng thành nhờ bị phá

Nhìn lại quá trình, lớp trừu tượng không xuất hiện hoàn chỉnh một lần.

Nó tiến hóa như sau:

```text
ban đầu
“dữ liệu đang ở trong bộ nhớ” là đủ
```

rồi:

```text
EXEC148
↓
cần biết cách tạo + vòng đời
```

sau đó:

```text
representation bắt buộc
↓
không phải lúc nào cũng được quyết định bằng ngưỡng tái sử dụng
```

rồi:

```text
cơ chế Split-K32 trực tiếp
↓
sẵn sàng thực thi không thể đồng nhất với trạng thái cư trú
```

Mỗi bước là một phản ví dụ.

Mỗi phản ví dụ không phá dự án.

Nó làm mô hình chính xác hơn.

Đây là một cách rất khác để nghĩ về thiết kế phần mềm.

Ta không cố đoán trước lớp trừu tượng hoàn hảo.

Ta để:

> **những trường hợp thật liên tục thử phá mô hình hiện tại.**

Chỉ khi nó bị phá, ta mới thêm đúng khái niệm còn thiếu.

## Dữ liệu không còn chỉ là “các byte ở một địa chỉ”

Tới đây, cách runtime nhìn một tensor đã thay đổi rất xa so với đầu cuốn sách.

Ban đầu:

```text
tensor
↓
hình dạng
kiểu dữ liệu
các byte
```

Rồi:

```text
tensor
↓
đang ở trong vùng bộ nhớ GPU có thể dùng
```

Bây giờ:

```text
tensor logic
↓
có thể có nhiều cách biểu diễn
↓
mỗi cách có danh tính riêng
↓
có thể đang hoặc chưa ở trong bộ nhớ
↓
có thể cần được tạo hoặc thu nhận
↓
có vòng đời riêng
↓
đường thực thi có thể tồn tại
↓
nhưng còn phải sẵn sàng cho yêu cầu hiện tại
```

Điều này có vẻ phức tạp hơn.

Nhưng đó không phải sự phức tạp do ta thích một hệ thống lớn.

Nó là sự phức tạp mà bằng chứng buộc runtime phải nhìn thấy.

## Và đây là ranh giới dẫn sang runtime v4

Sau nhiều vòng phản ví dụ, sáu chiều bắt đầu ổn định:

```text
danh tính

khả dụng thực thi

sẵn sàng thực thi

trạng thái cư trú

quá trình thu nhận

vòng đời
```

Đây chưa phải tuyên bố:

> “Sáu thứ này là mô hình phổ quát cho mọi runtime AI.”

Bằng chứng không cho phép nói vậy.

Điều có thể nói nhỏ hơn:

> **Sáu chiều này đủ để biểu diễn chính xác những họ cơ chế mà ArcLLM đã thực sự kiểm tra tới thời điểm đó, mà không cần nhét luật riêng của từng họ vào chính sách chung.**

Đó là câu chuyện của Chương 19.

### Nhớ 3 điều

1. **Có dữ liệu trong bộ nhớ không đồng nghĩa phép tính sẵn sàng chạy.** Trạng thái cư trú chỉ nên nói representation có tồn tại trong bộ nhớ hay không; trạng thái sẵn sàng thực thi phải được tách riêng.
2. **Không phải mọi cách tạo representation đều giống nhau.** EXEC148 là representation tùy chọn có thể chỉ đáng tạo khi tái sử dụng đủ lâu; một representation cần để phép tính khả thi thì phải được tạo bất kể mức tái sử dụng thấp hay chưa biết.
3. **Vòng đời là một quyết định độc lập.** Representation đã tạo có thể được giữ để tái sử dụng, bị loại khi hết hiệu lực hoặc khi chính sách bộ nhớ yêu cầu; việc đường thực thi tạm thời chưa sẵn sàng không tự động có nghĩa phải xóa dữ liệu.

**Chương 19 — Từ ArcLLM cụ thể tới một mô hình runtime tổng quát hơn**

Ta đã có sáu câu hỏi riêng:

```text
đây là gì?

có đường thực thi không?

có chạy được ngay không?

dữ liệu cần thiết đã tồn tại chưa?

nếu thiếu thì lấy hoặc tạo bằng cách nào?

sau đó giữ nó tới bao giờ?
```

Chương tiếp theo sẽ xem liệu sáu câu hỏi đó có thể trở thành một bề mặt runtime chung mà nhiều loại cơ chế khác nhau cùng đi qua hay không — mà không làm mất những ranh giới do chính các PASS và FAIL trước đó tạo ra.
