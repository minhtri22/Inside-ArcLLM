# Chương 2 — GGUF không còn là một file, nó trở thành tensor store

> **Mức đọc: Đi sâu**
>
> **Bản đồ xuyên suốt — đang mở: model file / tensor**
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
> ▶ **Đang mở ở chương này:** model file / tensor.


> **Câu hỏi của chương:** Một model có hàng tỷ con số được đặt trong file như thế nào, và runtime có phải bung tất cả chúng ra trước khi dùng không?

Ở cuối Chương 1, ArcLLM đã làm được một việc rất cơ bản nhưng quan trọng: xác nhận đúng file model, đọc được định dạng GGUF và nhìn thấy bên trong có **338 tensor**.

Con số 338 nói một điều đơn giản: file model không phải một khối bí ẩn duy nhất. Bên trong nó có nhiều “gói dữ liệu”, mỗi gói có tên, hình dạng, kiểu lưu trữ và vị trí riêng.

Chương này chỉ hỏi:

> **Bên trong chiếc hộp có gì, từng món nằm ở đâu, và có thể lấy đúng món cần dùng mà không phải đổ toàn bộ hộp ra sàn hay không?**

Ta vẫn chưa tính toán gì với model.

Chưa cần GPU chạy một phép nhân nào.

Việc trước mắt chỉ là **mở chiếc hộp cho đúng cách**.

## GGUF giống một kho hàng có mục lục

**GGUF là viết tắt của GGML Universal File** — một định dạng file nhị phân trong hệ sinh thái GGML, được dùng để lưu model cùng những thông tin cần thiết để runtime có thể đọc và sử dụng nó.

Model mà ArcLLM dùng trong giai đoạn này được lưu trong một file **GGUF**.

Có thể hình dung GGUF giống một kho hàng được sắp xếp khá cẩn thận.

Ở đầu kho có phần thông tin mô tả: đây là model thuộc kiến trúc nào, có bao nhiêu tensor, một số thông số chung là gì.

Sau đó là một danh mục cho biết từng tensor tên gì, có kích thước ra sao, dùng kiểu dữ liệu nào và nằm ở vị trí nào trong file.

Cuối cùng mới tới phần “hàng thật”: những byte chứa dữ liệu của model.

Có thể hình dung:

```text
FILE GGUF

┌──────────────────────────────┐
│ Thông tin chung / metadata   │
├──────────────────────────────┤
│ Danh mục tensor              │
│ tên / kích thước / kiểu      │
│ vị trí trong file            │
├──────────────────────────────┤
│                              │
│ Dữ liệu tensor               │
│                              │
└──────────────────────────────┘
```

**Metadata** đơn giản là “dữ liệu mô tả dữ liệu”.

Nếu một thùng hàng có nhãn:

```text
Khối lượng: 20 kg
Loại hàng: dễ vỡ
Kho: số 3
```

thì những dòng trên nhãn là metadata.

Còn những món thực sự nằm trong thùng mới là dữ liệu chính.

GGUF cũng gần như vậy.

Nhờ có phần mô tả này, runtime không phải nhìn vào byte thứ một triệu trong file rồi đoán:

> “Không biết số này thuộc phần nào của model?”

Nó có một bản đồ.

## Tensor thực ra là gì?

Ở chương trước, tôi tạm gọi tensor là “một bảng số”.

Bây giờ ta có thể làm rõ hơn một chút.

Trong quá trình huấn luyện, model học bằng cách điều chỉnh rất nhiều con số. Những con số đã học ấy thường được gọi là **weights — trọng số**.

Nếu hàng tỷ con số chỉ nằm trong một danh sách dài vô tận, việc quản lý chúng sẽ rất khó.

Vì vậy chúng được sắp xếp thành những khối có hình dạng rõ ràng.

Một dãy số có thể trông như:

```text
[0.2, -0.7, 1.1, 0.4]
```

Một bảng hai chiều có thể là:

```text
[ 0.2  -0.7 ]
[ 1.1   0.4 ]
```

**Tensor** là cách gọi tổng quát cho những khối số như vậy, kể cả khi chúng có nhiều hơn hai chiều.

Người đọc chưa cần học đại số tuyến tính để tiếp tục.

Hiện tại chỉ cần nhớ:

> **Tensor là một khối số có hình dạng xác định; model sử dụng những khối số đó trong các phép tính.**

Ở P1 — bước tiếp theo sau P0 — ArcLLM chưa cần hiểu toàn bộ ý nghĩa toán học của từng tensor.

Nhiệm vụ trước mắt đơn giản hơn nhiều:

> **Đếm đúng chúng và biết chính xác mỗi tensor nằm ở đâu trong file.**

Kết quả của file đã được khóa từ P0 là:

```text
F32   : 141 tensor
Q4_K  : 168 tensor
Q6_K  :  29 tensor
-------------------
Tổng  : 338 tensor
```

Ta có thể kiểm tra ngay:

```text
141 + 168 + 29 = 338
```

Nhưng F32, Q4_K và Q6_K là gì?

## Không phải mọi con số đều được lưu giống nhau

Hãy bắt đầu với loại dễ hiểu nhất: **F32**.

F32 là cách viết ngắn của **32-bit floating point — số thực dấu phẩy động 32 bit**.

Một giá trị F32 dùng 32 bit.

Mà:

```text
8 bit = 1 byte
```

nên:

```text
32 bit = 4 byte
```

Giả sử ta có 256 giá trị và lưu tất cả bằng F32:

```text
256 × 4 byte
= 1.024 byte
```

Chỉ 256 con số đã cần 1.024 byte.

Với một model có hàng tỷ trọng số, con số ấy tăng lên rất nhanh.

Đó là một trong những lý do người ta sử dụng **quantization — lượng tử hóa**.

Ý tưởng cơ bản không quá khó.

Thay vì lưu mỗi trọng số với độ chi tiết rất cao, ta tìm cách biểu diễn nó bằng ít bit hơn. Đổi lại, ta chấp nhận một mức xấp xỉ có kiểm soát.

Mục tiêu là giảm dung lượng và giảm lượng dữ liệu phải di chuyển.

Trong file đang được ArcLLM nghiên cứu, hai dạng quan trọng là **Q4_K** và **Q6_K**.

Ở đây ta chưa cần học cấu trúc chi tiết của chúng. Chỉ cần biết đây là hai cách đóng gói trọng số theo từng khối.

Trong phép tính kích thước mà P1 sử dụng, một khối gồm 256 giá trị chiếm:

```text
F32   : 1.024 byte
Q4_K  :   144 byte
Q6_K  :   210 byte
```

Có thể một câu hỏi lập tức xuất hiện:

> Nếu Q4 có chữ “4”, tại sao 256 giá trị không phải chỉ cần 128 byte?

Ta thử tính:

```text
256 × 4 bit
= 1.024 bit
= 128 byte
```

Phép tính đó không sai.

Nhưng một block Q4_K thực tế không chỉ chứa những bit đại diện cho trọng số. Nó còn cần thêm thông tin để sau này có thể giải mã những giá trị ấy một cách có ý nghĩa.

Vì vậy block thực tế trong trường hợp này là **144 byte**, không phải 128 byte.

Đây là một chi tiết nhỏ nhưng rất hữu ích:

> **“4 bit” không có nghĩa toàn bộ cấu trúc lưu trữ thực tế chỉ tốn đúng 4 bit cho mỗi giá trị.**

Format còn có chi phí phụ.

Dù vậy:

```text
144 byte < 1.024 byte
```

Khoảng cách vẫn rất lớn.

Đó là lý do giữ trọng số trong dạng đã được lượng tử hóa có giá trị.

## Tại sao không bung toàn bộ model thành F32 ngay từ đầu?

Hãy tưởng tượng bạn mua một chiếc tủ được đóng trong vài hộp phẳng.

Một cách làm là vừa nhận hàng đã mở tất cả hộp, lắp mọi bộ phận rồi trải chúng kín căn phòng, dù chưa biết khi nào cần đến từng món.

Cách khác là giữ mọi thứ gọn trong dạng đóng gói, và lấy đúng phần cần dùng khi cần.

P1 chọn tư duy thứ hai.

Nếu Q4_K trong file đã được đóng gói rất gọn, runtime không nên bắt đầu bằng việc mở toàn bộ chúng thành F32 rồi tạo thêm một bản sao lớn trong RAM.

ArcLLM muốn:

> **Truy cập trực tiếp những byte Q4_K đang nằm trong GGUF.**

Đây là lúc xuất hiện hai khái niệm mới:

**memory mapping** và **zero-copy view**.

Tên nghe khá kỹ thuật, nhưng ý tưởng lại rất đời thường.

## Thay vì bê cả kho vào nhà, hãy mở một cánh cửa nhìn vào kho

Một cách dễ nghĩ khi đọc file là:

```text
Mở file
   ↓
Đọc toàn bộ file
   ↓
Chép tất cả vào một vùng RAM mới
   ↓
Bắt đầu sử dụng
```

Cách đó có thể hoạt động.

Nhưng với file model lớn, nó có nghĩa ta vừa có file gốc, vừa tạo thêm một vùng nhớ lớn để chứa bản sao của file.

P1 dùng một cơ chế của hệ điều hành gọi là **memory mapping — ánh xạ file vào không gian bộ nhớ của chương trình**.

Có thể hình dung hệ điều hành mở cho ArcLLM một “cửa sổ” nhìn vào file.

Runtime có thể truy cập một vùng trong file gần giống như đang truy cập bộ nhớ, thay vì tự đọc toàn bộ file rồi chép nó sang một buffer khác.

Trong P1, cửa sổ này là **read-only — chỉ đọc**.

ArcLLM không được phép dùng nó để sửa file model.

Ánh xạ chỉ đọc giúp tránh ghi đè model gốc và cho phép dùng GGUF như một:

> **tensor store — kho tensor.**

Runtime biết tensor mình cần nằm ở đâu, rồi truy cập đúng vùng byte tương ứng.

P1 gọi cách truy cập này là **direct zero-copy view**.

Nhưng chữ “zero-copy” rất dễ làm người đọc tưởng tượng quá xa.

Nó **không có nghĩa** dữ liệu từ ổ đĩa bằng cách nào đó bay thẳng vào phép tính mà không liên quan tới RAM.

Hệ điều hành vẫn quản lý việc đưa những phần dữ liệu cần thiết từ file vào bộ nhớ vật lý.

“Zero-copy” trong phạm vi P1 có nghĩa hẹp hơn:

> **ArcLLM không tự tạo thêm một bản sao toàn bộ payload chỉ để có thể đọc tensor.**

Đó mới là điều P1 thực sự chứng minh.

## Biết tensor nằm ở đâu vẫn chưa đủ

Giả sử danh mục nói một tensor bắt đầu tại byte số 1.000 và dài 144 byte.

Ta có:

```text
Bắt đầu : 1.000
Độ dài  :   144
Kết thúc: 1.144
```

Nếu tensor tiếp theo bắt đầu ở byte 1.144, hai tensor đứng sát nhau nhưng không đè lên nhau.

```text
Tensor A
1.000 ───────────── 1.144

Tensor B
                    1.144 ──────────
```

Nhưng nếu tensor B lại bắt đầu ở 1.130:

```text
Tensor A
1.000 ───────────── 1.144
                 █████
Tensor B        1.130 ──────────
```

một đoạn byte đang bị cả hai tensor cùng nhận là của mình.

Đó gọi là **overlap — chồng lấn**.

P1 vì vậy kiểm tra hai điều.

Thứ nhất là **bounds — giới hạn**.

Nếu file kết thúc ở byte 10.000 nhưng một tensor tuyên bố dữ liệu của nó kéo dài tới byte 10.200, rõ ràng có vấn đề.

Thứ hai là overlap.

Hai tensor không được vô tình trỏ vào những vùng dữ liệu chồng lên nhau.

Kết quả P1 cho file thật:

```text
338 tensor
→ tất cả nằm trong bounds hợp lệ
→ không có tensor overlap
```

Điều này không hào nhoáng.

Nhưng trước khi cho GPU tính hàng triệu phép toán, runtime phải chắc rằng mình đang đọc đúng byte.

Nếu địa chỉ sai, một chương trình chạy nhanh hơn chỉ có nghĩa là:

> **Ta nhận được kết quả sai nhanh hơn.**

## Q4_K được giữ nguyên dạng đóng gói

Một mục tiêu quan trọng khác của P1 là xác nhận ArcLLM có thể đi thẳng tới dữ liệu **Q4_K packed — Q4_K vẫn còn nguyên dạng đóng gói trong file**.

P1 đã PASS điều đó.

Nói đơn giản, runtime có thể đi theo chuỗi:

```text
Cần tensor nào?
      ↓
Tensor nằm ở offset nào?
      ↓
Nó dài bao nhiêu byte?
      ↓
Kiểu dữ liệu là Q4_K
      ↓
Truy cập trực tiếp vùng byte đó
```

mà chưa cần mở toàn bộ tensor thành một mảng F32 mới.

Điều này đặt nền cho những bước sau.

Nhưng ta phải giữ ranh giới kết luận thật rõ.

P1 **chưa chứng minh GPU tính được Q4_K**.

P1 chưa chứng minh model tạo ra câu trả lời đúng.

P1 chưa đo tốc độ.

P1 cũng chưa chứng minh toàn bộ model có thể chạy.

Nó chỉ chứng minh một điều:

> **Lớp lưu trữ đã đủ đáng tin để bước tiếp.**

## Hai lần dừng mà chưa phải FAIL khoa học

P1 còn để lại một bài học rất đáng giữ.

Lần chạy đầu tiên không đi tới phép thử.

Một script PowerShell gặp lỗi khi phân tích cú pháp.

Sau khi sửa lỗi đó, lần tiếp theo đi xa hơn nhưng quá trình build lại dừng vì một xung đột tên `max` trên Windows.

Cả hai lần đều chưa chạy xong.

Nhưng trong lịch sử nghiên cứu ArcLLM, chúng **không được coi là bằng chứng P1 FAIL**.

Tại sao?

Bởi câu hỏi P1 là:

> “Ta có thể ánh xạ GGUF, xác định đúng từng tensor và truy cập trực tiếp Q4_K hay không?”

Ở hai lần lỗi trước, câu hỏi đó chưa hề được đem ra kiểm tra.

Không có kết quả âm tính.

Thậm chí chưa có kết quả.

Đây là sự khác nhau rất quan trọng giữa:

```text
Phép thử đã chạy
      ↓
kết quả không đạt
      ↓
FAIL khoa học
```

và:

```text
Phép thử chưa chạy được
      ↓
lỗi công cụ / build / hạ tầng
      ↓
chưa có bằng chứng khoa học
```

Sau khi hai lỗi hạ tầng được sửa mà không thay đổi câu hỏi của P1, phép thử mới thực sự chạy.

Và lần này P1 PASS.

Đây cũng là một nguyên tắc mà chúng ta sẽ gặp lại nhiều lần trong cuốn sách:

> **Một chương trình báo lỗi không tự động có nghĩa giả thuyết sai.**

Muốn kết luận điều gì, trước hết phải chắc rằng thứ cần kiểm tra đã thực sự được kiểm tra.

## P1 đã chứng minh được gì?

Đến cuối P1, ArcLLM biết rằng file GGUF đã khóa từ P0 có thể được dùng như một **tensor store chỉ đọc**.

338 tensor được nhận diện:

```text
141 F32
168 Q4_K
 29 Q6_K
```

Tất cả vùng byte đều nằm trong giới hạn file.

Không tensor nào chồng lên tensor khác.

Dữ liệu Q4_K có thể được truy cập trực tiếp trong khi vẫn giữ dạng packed.

Đó là tất cả những gì P1 được phép nói — nhưng từng ấy đã đủ để mở câu hỏi tiếp theo.

Ta đã có bản đồ kho hàng.

Ta biết từng kiện nằm ở đâu.

Ta có thể nhìn trực tiếp vào dữ liệu mà không cần bung toàn bộ kho thành một bản F32 mới trong RAM.

Nhưng dữ liệu ấy **chưa nằm trong một môi trường mà GPU của ArcLLM có thể thực sự sử dụng để làm việc**.

Ta mới mở được kho.

Chưa xây nhà máy.

Câu hỏi tiếp theo vì thế trở nên tự nhiên:

> **Làm thế nào đưa dữ liệu tới GPU, giữ nó ở đó và tạo một môi trường để GPU có thể thật sự nhận một công việc rồi báo rằng công việc đã hoàn thành?**

Đó là P2.

### Nhớ 3 điều

1. **GGUF không chỉ là một “file model”.** Nó chứa thông tin mô tả, danh mục tensor và dữ liệu giúp runtime biết chính xác từng tensor nằm ở đâu.
2. **Quantization giúp lưu trọng số gọn hơn.** P1 giữ Q4_K và Q6_K ở dạng đóng gói thay vì mở toàn bộ thành F32 ngay từ đầu.
3. **Script hỏng hoặc build hỏng không tự động là FAIL của giả thuyết.** Chỉ khi phép thử thực sự chạy, evidence mới được quyền trả lời câu hỏi.

**Chương 3 — Xây phần lõi Vulkan**

Ở Chương 2, dữ liệu vẫn chủ yếu nằm phía file và bộ nhớ do hệ điều hành quản lý.

Chương tiếp theo sẽ đưa chúng ta sang phía GPU: device là gì, queue là gì, vì sao cần những vùng bộ nhớ tồn tại lâu dài, và làm thế nào để biết GPU đã thật sự nhận và hoàn thành một công việc.
