# Phần 0 — Bản đồ trước khi vào rừng

> **Mức đọc: Nền tảng**
>
> Phần này dành cho người bắt đầu từ số 0. Nếu bạn chưa biết model, LLM, parameter, dense, Transformer, CPU hay GPU là gì, hãy đọc phần này trước Chương 1.

Cuốn sách này sẽ đi rất sâu vào bên trong một runtime LLM. Nhưng trước khi mở cỗ máy, ta cần một tấm bản đồ.

Mục tiêu của Phần 0 không phải biến bạn thành kỹ sư AI. Ta chỉ cần dựng đủ những “móc” kiến thức để khi các từ như **token**, **tensor**, **decoder layer**, **GPU** hay **quantization** xuất hiện ở những chương sau, bạn biết chúng đang nằm ở đâu trong bức tranh lớn.

Nếu có chỗ nào chưa nhớ ngay, điều đó hoàn toàn bình thường. Hãy giữ lại sơ đồ lớn trước. Những chi tiết sẽ được mở dần khi ArcLLM thật sự cần tới chúng.

## Hai bản đồ cần nhớ

Bản đồ thứ nhất trả lời:

> **LLM nằm ở đâu trong thế giới AI?**

Bản đồ thứ hai trả lời:

> **Một đoạn văn đi qua model như thế nào để tạo ra token tiếp theo?**

Có thể đặt hai bản đồ cạnh nhau như sau:

```text
HỌ HÀNG KHÁI NIỆM                         ĐƯỜNG ĐI CỦA DỮ LIỆU

AI                                        Văn bản người dùng
↓                                         ↓
Machine Learning                          Tokenizer
↓                                         ↓
Neural Network                            Token / token ID
↓                                         ↓
Language Model                            Embedding → tensor
↓                                         ↓
LLM                                       Decoder-only Transformer
↓                                         ↓
Transformer                               Nhiều decoder layer
↓                                         ├─ RMSNorm
Decoder-only Transformer                  ├─ Attention
↓                                         ├─ FFN
Nhiều decoder layer                       └─ Residual
↓                                         ↓
Parameters / Weights                      Logits
                                          ↓
                                          Token tiếp theo
                                          ↓
                                          lặp lại
```

Đừng cố học thuộc sơ đồ này ngay. Chỉ cần biết mỗi chương sau sẽ mở một phần của nó.

## AI không đồng nghĩa với LLM

**Artificial Intelligence — trí tuệ nhân tạo (AI)** là tên rất rộng cho những hệ thống thực hiện các công việc mà ta thường liên hệ với khả năng “thông minh”: nhận biết, dự đoán, lập kế hoạch, tạo nội dung hoặc ra quyết định.

Bên trong chiếc ô rất rộng đó có **Machine Learning — học máy**.

Học máy là cách xây hệ thống không chỉ bằng những luật viết tay kiểu:

```text
nếu A thì làm B
nếu C thì làm D
```

mà còn bằng cách cho hệ thống học những mẫu từ dữ liệu.

Một họ rất quan trọng của học máy là **Neural Network — mạng nơ-ron nhân tạo**. Đây là những hệ thống gồm nhiều phép biến đổi số học nối tiếp nhau. Từ “nơ-ron” gợi liên tưởng tới sinh học, nhưng trong cuốn sách này hãy hiểu nó đơn giản là **một mô hình toán học được tạo bởi rất nhiều phép tính và rất nhiều con số có thể học được**.

Một **Language Model — mô hình ngôn ngữ** là model được dùng để mô hình hóa ngôn ngữ. Một cách rất hữu ích để hiểu nó là:

> **Cho những token đã có, model ước lượng token nào có thể đứng tiếp theo.**

Khi mô hình ngôn ngữ có quy mô rất lớn, ta thường gọi nó là:

**Large Language Model — mô hình ngôn ngữ lớn (LLM).**

Vì vậy:

```text
AI
└─ Machine Learning
   └─ Neural Network
      └─ Language Model
         └─ Large Language Model
```

LLM là một phần của thế giới AI. Nó không đồng nghĩa với toàn bộ AI.

## Model là gì?

Trong đời thường, ta có thể gọi “model” là mô hình.

Trong cuốn sách này, **model — mô hình** là một hệ thống đã học được một lượng lớn các con số và cấu trúc để biến đầu vào thành đầu ra.

Đối với một LLM, đầu vào cuối cùng không phải là câu chữ theo cách con người nhìn thấy. Văn bản sẽ được mã hóa thành các đơn vị số, đi qua nhiều phép tính, rồi model tạo điểm cho những token có thể xuất hiện tiếp theo.

Điều rất quan trọng là:

> **Model không phải ứng dụng chat.**

Một ứng dụng như ChatGPT có thể có giao diện, lịch sử hội thoại, công cụ, tìm kiếm, lưu tệp và nhiều lớp khác. Model chỉ là một thành phần bên trong hệ thống lớn hơn.

Ta sẽ dùng ranh giới này rất nhiều:

```text
Ứng dụng
   ↓
Model
   ↓
Runtime
   ↓
Phần cứng
```

## Parameter và weight: model đã học cái gì?

Trong quá trình **training — huấn luyện**, model được điều chỉnh để làm tốt hơn một nhiệm vụ. Với LLM, một nhiệm vụ nền tảng là dự đoán token tiếp theo.

Những con số có thể được điều chỉnh trong quá trình học được gọi chung là:

**parameter — tham số học được của model.**

Một loại parameter rất phổ biến là:

**weight — trọng số.**

Ở mức nhập môn, bạn có thể hình dung:

```text
training
   ↓
điều chỉnh rất nhiều con số
   ↓
parameters / weights
   ↓
lưu chúng vào model
```

Sau khi huấn luyện xong, khi ta dùng model để xử lý một yêu cầu mới, quá trình đó gọi là:

**inference — suy luận.**

Ta có hai giai đoạn khác nhau:

```text
TRAINING
học / điều chỉnh parameters
        ↓
model đã huấn luyện
        ↓
INFERENCE
dùng parameters đã học để xử lý đầu vào mới
```

ArcLLM tập trung vào **inference**, không phải huấn luyện model từ đầu.

## 1.5B, 7B nghĩa là gì?

Trong tên model, bạn có thể gặp:

```text
1.5B
7B
14B
```

Chữ **B** ở đây là **billion — tỷ**.

Một model “1.5B” thường có khoảng 1,5 tỷ parameter theo cách nhà phát hành đặt tên cho quy mô model.

Một model “7B” thường ở cỡ khoảng 7 tỷ parameter.

Con số này **không phải kích thước file tính bằng byte**.

Cùng một model có thể được lưu bằng nhiều cách khác nhau. Cách lưu mỗi parameter bằng bao nhiêu bit sẽ ảnh hưởng rất lớn tới dung lượng file và lượng bộ nhớ cần thiết khi chạy.

Ta sẽ mở chuyện này ở Chương 2.

## Dense là gì?

Một từ rất hay xuất hiện khi nói về model là:

**dense model — model đặc.**

Ở mức cần thiết cho cuốn sách này, hãy hiểu như sau:

> Trong một model dense truyền thống, mỗi token đi qua cùng chuỗi layer chính của model.

Ví dụ:

```text
token
  ↓
Layer 0
  ↓
Layer 1
  ↓
Layer 2
  ↓
...
```

Một hướng khác là **Mixture of Experts (MoE) — hỗn hợp chuyên gia**. Model có nhiều nhánh “expert”, nhưng mỗi token có thể chỉ được định tuyến qua một phần trong số đó.

Điều cần nhớ là:

```text
dense / MoE
→ nói về cách kiến trúc model tổ chức việc tính toán

F32 / Q4 / Q6
→ nói về cách các con số được biểu diễn và lưu trữ
```

Hai câu chuyện này khác nhau.

ArcLLM trong hành trình chính của cuốn sách làm việc với một model decoder dạng dense.

## Transformer là gì?

Phần lớn LLM hiện đại thuộc họ kiến trúc **Transformer**.

Ta chưa cần học công thức Transformer.

Ta chỉ cần một bức tranh đủ để không bị lạc khi các chương sau nói tới attention, FFN hay decoder layer.

Một Transformer có thể được xây từ nhiều **layer — lớp xử lý** nối tiếp nhau.

Model mà ArcLLM nghiên cứu thuộc loại:

**decoder-only Transformer — Transformer chỉ dùng khối decoder để sinh token tiếp theo.**

Có thể hình dung:

```text
token / tín hiệu đầu vào
        ↓
Decoder layer 0
        ↓
Decoder layer 1
        ↓
Decoder layer 2
        ↓
...
        ↓
Decoder layer N
        ↓
điểm cho token tiếp theo
```

Trong một decoder layer, ta sẽ gặp những khối như:

```text
tín hiệu vào
    ↓
RMSNorm
    ↓
Attention
    ↓
Residual
    ↓
RMSNorm
    ↓
FFN
    ↓
Residual
    ↓
tín hiệu ra
```

Bạn chưa cần hiểu công thức của từng khối.

Chỉ cần nhớ:

- **RMSNorm — phép chuẩn hóa** giúp giữ tín hiệu ở thang phù hợp.
- **Attention — cơ chế chú ý** giúp vị trí hiện tại kết hợp thông tin từ các vị trí khác.
- **FFN — Feed-Forward Network, nhánh biến đổi tín hiệu** xử lý mỗi vị trí qua một nhóm phép biến đổi khác.
- **Residual — đường cộng tắt** đưa một phần tín hiệu cũ cộng trở lại tín hiệu mới.

Chương 4 và 5 sẽ mở những khối này đúng lúc chúng ta thật sự cần chạy chúng.

## Token là gì?

Con người nhìn thấy:

```text
Xin chào!
```

Model không xử lý câu đó như một “ý nghĩa nguyên khối”.

Một **tokenizer — bộ tách và mã hóa văn bản** biến văn bản thành những đơn vị gọi là:

**token — đơn vị mà model dùng để xử lý văn bản.**

Một token không nhất thiết là một từ.

Nó có thể là:

- một ký tự;
- một phần của từ;
- cả một từ;
- dấu câu;
- hoặc một chuỗi ký tự khác tùy tokenizer.

Mỗi token sau đó được gán một con số:

**token ID — mã số của token.**

Ví dụ minh họa:

```text
"Xin chào!"
    ↓
tokenizer
    ↓
["Xin", " chào", "!"]
    ↓
[314, 9821, 17]
```

Các con số trên chỉ là ví dụ. Tokenizer thật của từng model có thể chia khác hoàn toàn.

## Từ token ID tới tensor

Token ID chỉ là một số nguyên.

Model cần biến nó thành một dãy số mà các layer có thể tính toán.

Bước đó dùng **embedding — cách biến một ID thành một vector số**.

Ví dụ:

```text
token ID = 314
      ↓
embedding
      ↓
[0.12, -0.08, 0.44, ...]
```

Dãy số này là một ví dụ của **tensor — khối số có hình dạng xác định**.

Tensor có thể là một dãy một chiều, một bảng hai chiều hoặc có nhiều chiều hơn.

Trong model, cả:

- trọng số đã học;
- dữ liệu đang chảy qua model;
- trạng thái trung gian;

đều có thể được biểu diễn bằng tensor.

Chương 2 sẽ mở file model và nhìn tensor thật nằm trong đó.

## CPU, GPU và bộ nhớ là gì?

Model chứa rất nhiều phép tính.

Nhưng file model nằm trên ổ đĩa không tự tính được.

Ta cần phần cứng.

**CPU — Central Processing Unit, bộ xử lý trung tâm** là bộ xử lý đa dụng của máy tính. Nó rất linh hoạt và giỏi điều phối nhiều loại công việc.

**GPU — Graphics Processing Unit, bộ xử lý đồ họa** ban đầu nổi tiếng nhờ xử lý đồ họa, nhưng cấu trúc của nó cũng rất phù hợp với việc thực hiện nhiều phép toán số học tương tự nhau song song.

LLM phải thực hiện rất nhiều phép nhân và cộng trên những khối số lớn. Vì vậy GPU thường rất quan trọng khi chạy model.

Ta cũng cần phân biệt nơi dữ liệu sống:

- **storage — ổ lưu trữ** như SSD: giữ file lâu dài;
- **memory — bộ nhớ làm việc** như RAM hoặc vùng bộ nhớ GPU có thể truy cập: giữ dữ liệu đang cần dùng;
- **cache — bộ nhớ đệm**: giữ lại dữ liệu hoặc kết quả hữu ích để tránh làm lại công việc.

Một bức tranh đơn giản:

```text
SSD
giữ file model
   ↓
RAM / vùng nhớ runtime
đưa dữ liệu vào trạng thái làm việc
   ↓
CPU / GPU
thực hiện phép tính
```

Máy ArcLLM ban đầu dùng GPU tích hợp Intel Arc 140V. Trên loại máy dùng bộ nhớ hợp nhất, CPU và GPU có thể cùng truy cập một phần không gian bộ nhớ vật lý. Những chi tiết chính xác sẽ được nói ở đúng chương cần tới chúng.

## Runtime là gì?

Giữa model và phần cứng cần một lớp tổ chức mọi thứ.

Lớp đó là:

**runtime — hệ thực thi model.**

Runtime phải làm những việc như:

- đọc file model;
- hiểu tensor nào nằm ở đâu;
- chuẩn bị bộ nhớ;
- gửi phép tính xuống CPU hoặc GPU;
- giữ lại dữ liệu cần dùng tiếp;
- chạy các layer theo đúng thứ tự;
- tạo token mới;
- lặp lại quá trình.

Có thể hình dung:

```text
MODEL = bản thiết kế + các con số đã học

RUNTIME = bộ máy đọc bản thiết kế
          và biến nó thành công việc thật

CPU/GPU = nơi phép tính thực sự xảy ra
```

ArcLLM là câu chuyện xây lớp **runtime** đó từ những nguyên lý đầu tiên.

## Quantization là gì?

Nếu model có hàng tỷ parameter, cách lưu mỗi con số trở nên rất quan trọng.

**quantization — lượng tử hóa** là kỹ thuật biểu diễn trọng số bằng ít bit hơn để giảm dung lượng và lượng dữ liệu phải di chuyển, đổi lại chấp nhận một mức xấp xỉ có kiểm soát.

Ví dụ ở mức trực giác:

```text
nhiều bit hơn
→ mô tả số chi tiết hơn
→ thường tốn nhiều bộ nhớ hơn

ít bit hơn
→ lưu gọn hơn
→ phải có cách mã hóa / giải mã phù hợp
```

Trong sách bạn sẽ gặp F32, Q4_K và Q6_K.

Chưa cần nhớ chúng ngay. Chương 2 sẽ bắt đầu từ bit và byte rồi giải thích từng bước.

## Một token đi qua toàn hệ thống như thế nào?

Bây giờ ta có thể ghép các khái niệm lại.

Đây là **sơ đồ xuyên suốt của cuốn sách**:

```text
VĂN BẢN NGƯỜI DÙNG
        ↓
TOKENIZER
        ↓
TOKEN / TOKEN ID
        ↓
EMBEDDING → TENSOR
        ↓
┌───────────────────────────────────────┐
│ MODEL: PARAMETERS / WEIGHTS           │
│                                       │
│  Decoder layer 0                      │
│      RMSNorm → Attention → FFN        │
│               ↓                       │
│  Decoder layer 1                      │
│               ↓                       │
│            ...                        │
│               ↓                       │
│  Decoder layer N                      │
└───────────────────────────────────────┘
        ↓
LOGITS — điểm cho các token ứng viên
        ↓
CHỌN TOKEN TIẾP THEO
        ↓
KV CACHE giữ trạng thái hữu ích
        ↓
LẶP LẠI

Trong toàn bộ đường trên:

RUNTIME
   ↓
tổ chức tensor + bộ nhớ + thứ tự công việc
   ↓
CPU / GPU
   ↓
thực hiện các phép tính thật
```

Và nếu nhìn theo hành trình của ArcLLM:

```text
model file
    ↓
tensor
    ↓
runtime / GPU / memory
    ↓
RMSNorm / attention / FFN
    ↓
một decoder layer
    ↓
toàn bộ các decoder layer
    ↓
KV cache + sinh nhiều token
    ↓
benchmark + tối ưu
    ↓
representation + lifecycle
    ↓
một runtime có kiến trúc rõ ràng hơn
```

Từ Chương 1 trở đi, đầu mỗi chương sẽ có một bản rút gọn của sơ đồ này và đánh dấu phần đang được mở.

## Bạn cần nhớ gì trước khi vào Chương 1?

Chỉ cần giữ chín ý:

1. **AI** là chiếc ô lớn; **LLM** chỉ là một loại model trong thế giới AI.
2. **Model** chứa cấu trúc và những con số đã học.
3. Những con số model học được gọi chung là **parameters**; **weights** là một loại parameter quan trọng.
4. **Dense** mô tả cách kiến trúc dùng các layer; nó không đồng nghĩa với quantization.
5. **Transformer** là họ kiến trúc; model trong hành trình này là một **decoder-only Transformer**.
6. Văn bản được đổi thành **token**, rồi thành những **tensor** số.
7. Tensor đi qua nhiều **decoder layer** chứa các khối như RMSNorm, attention và FFN.
8. **Runtime** tổ chức việc đọc model, bộ nhớ và thực thi; **CPU/GPU** thực hiện phép tính thật.
9. ArcLLM nghiên cứu **inference — suy luận**, không phải huấn luyện LLM từ đầu.

Nếu bạn hiểu được bức tranh này dù chưa nhớ hết tên, bạn đã đủ nền để bước vào Chương 1.

**Tiếp theo: [Chương 1 — Bên dưới một câu trả lời AI có gì?](01-khoa-target-truoc-khi-toi-uu.md)**
