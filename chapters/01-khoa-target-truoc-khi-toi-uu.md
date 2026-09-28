# Chương 1 — Bên dưới một câu trả lời AI có gì?

Khi chúng ta mở một ứng dụng AI, gõ một câu hỏi rồi vài giây sau nhận được câu trả lời, mọi thứ trông rất đơn giản.

Nhưng bên dưới ô chat ấy là nhiều lớp khác nhau.

Ứng dụng có thể lo giao diện, lưu lịch sử trò chuyện, tìm tài liệu, gọi công cụ hoặc kết nối Internet. Một **model** lại làm công việc khác: nó chứa những cấu trúc và hàng tỷ con số đã học được trong quá trình huấn luyện, rồi dùng chúng để dự đoán phần tiếp theo của văn bản.

Giữa model và phần cứng còn có một lớp rất quan trọng: **runtime**.

Có thể hình dung đơn giản như thế này:

```text
Người dùng
    ↓
Ứng dụng AI  ↔  Ứng dụng khác / Tools
    ↓
Model
    ↓
Runtime
    ↓
CPU / GPU / bộ nhớ
```

Sơ đồ này cố tình được đơn giản hóa. Trong một hệ thống thật, dữ liệu có thể đi qua lại giữa các lớp nhiều lần. Ứng dụng có thể yêu cầu một công cụ tìm kiếm thông tin, đưa kết quả trở lại model, rồi tiếp tục sinh câu trả lời. Nhưng ở thời điểm này, chúng ta chỉ cần nhớ một điều: **ArcLLM nằm ở phần dưới của sơ đồ, gần model và phần cứng hơn là gần giao diện người dùng.**

Runtime có thể hiểu là **hệ thực thi model**.

Nếu model là một bản thiết kế khổng lồ thì runtime là bộ máy đọc bản thiết kế đó và biến nó thành công việc thật trên máy tính.

Một file model nằm yên trên ổ đĩa không thể tự yêu cầu GPU thực hiện phép nhân, không tự biết phải giữ dữ liệu nào trong bộ nhớ, cũng không tự biết kết quả của phép tính này phải được chuyển sang phép tính tiếp theo ra sao.

Runtime làm những việc ấy.

Nó đọc model, chuẩn bị dữ liệu, tổ chức bộ nhớ, quyết định phần việc nào được thực hiện ở đâu, gửi công việc xuống phần cứng rồi thu kết quả về để tiếp tục bước kế tiếp.

Và quá trình đó lặp đi lặp lại cho tới khi model tạo được câu trả lời.

## Token không nhất thiết là một chữ

Ở đây chúng ta cần làm rõ một từ sẽ xuất hiện rất nhiều trong cuốn sách này: **token**.

Có một cách giải thích thường gặp là “token là một mảnh văn bản”. Cách nói ấy dễ nhớ, nhưng nếu hiểu quá sát thì lại không chính xác.

Token không cố định là một ký tự.

Nó cũng không nhất thiết là một từ.

Một token là **một đơn vị mà model dùng để xử lý văn bản**. Trước khi văn bản được đưa vào model, một thành phần gọi là **tokenizer — bộ tách và mã hóa văn bản —** sẽ chuyển văn bản thành các token, rồi mỗi token được biểu diễn bằng một con số.

Tùy model và tokenizer, một token có thể tương ứng với một ký tự, một chuỗi nhiều ký tự, một phần của một từ, cả một từ, dấu câu hoặc thậm chí một phần liên quan đến khoảng trắng.

Ví dụ, hãy tưởng tượng ta có chữ:

```text
ChatGPT!
```

Một tokenizer có thể chia nó thành:

```text
Chat | GPT | !
```

Nhưng một tokenizer khác hoàn toàn có thể chia theo cách khác.

Vì vậy khi trong sách tôi nói “model sinh thêm một token”, hãy hiểu đơn giản là:

> Model vừa tạo thêm **một đơn vị văn bản theo cách mã hóa riêng của nó**, chứ không nhất thiết vừa tạo thêm một chữ hay một từ hoàn chỉnh.

Chúng ta chưa cần biết tokenizer hoạt động sâu bên trong như thế nào. Chỉ cần hiểu điều này là đủ để đi tiếp.

## Và đây là nơi ArcLLM xuất hiện

ArcLLM nghiên cứu chính lớp **runtime** vừa nói ở trên.

Tên của nó cũng bắt nguồn từ một câu chuyện rất đời thường.

Dự án không bắt đầu trong một trung tâm dữ liệu với hàng trăm GPU. Nó bắt đầu trên chiếc máy tính mà người phát triển đang có.

Máy đó sử dụng Intel Core Ultra 7 258V và GPU tích hợp **Intel Arc 140V**.

Đó cũng chính là nguồn gốc của cái tên:

```text
Intel Arc
    +
LLM
    ↓
ArcLLM
```

`Arc` đến từ dòng GPU Intel Arc trên chiếc máy phát triển ban đầu.

`LLM` là viết tắt của **Large Language Model — mô hình ngôn ngữ lớn**.

Cái tên vì thế giống một dấu mốc ghi lại nơi dự án được sinh ra hơn là một giới hạn kỹ thuật.

ArcLLM không có nghĩa là “một runtime chỉ được phép chạy trên Intel Arc”. Và ArcLLM cũng không được định nghĩa là runtime dành riêng cho một model duy nhất.

Trong nghiên cứu, chúng ta sẽ thường **khóa một model cụ thể** cho một thí nghiệm. Việc đó rất quan trọng: nếu vừa đổi runtime vừa đổi model thì khi kết quả thay đổi, chúng ta sẽ không biết nguyên nhân đến từ đâu.

Nhưng khóa model trong một thí nghiệm không đồng nghĩa với việc hard-code toàn bộ kiến trúc vào model đó.

Mục tiêu rộng hơn của ArcLLM là xây dựng và hiểu một **hệ thực thi model**, trong đó những phần phụ thuộc vào model, loại trọng số hay phần cứng có thể được tách ra rõ ràng.

Điều đó sẽ trở nên quan trọng hơn rất nhiều ở các chương sau.

Còn hiện tại, chỉ cần giữ một ranh giới thật rõ:

> **ArcLLM là runtime. Nó chưa phải là một hệ AI hoàn chỉnh.**

ArcLLM không phải giao diện chat.

Nó không phải một agent tự đi làm nhiều bước.

Nó chưa phải hệ thống gọi công cụ.

Nó không phải RAG — tức hệ thống đi tìm tài liệu bên ngoài rồi đưa tài liệu đó cho model.

Những lớp ấy có thể nằm phía trên runtime.

Nếu sau này một ứng dụng muốn dùng ArcLLM làm động cơ bên dưới thì đó là một bài toán khác.

Cuốn sách này tập trung vào câu hỏi thấp hơn:

> **Từ một file model nằm trên ổ đĩa, làm thế nào để những phép tính bên trong nó thực sự chạy trên CPU, GPU và bộ nhớ của một chiếc máy tính?**

## Nhưng llama.cpp và Ollama đã tồn tại rồi

Đến đây có một câu hỏi rất tự nhiên.

Nếu đã có `llama.cpp`, đã có Ollama, và chỉ cần vài câu lệnh là có thể chạy model trên máy cá nhân, tại sao lại mất công xây ArcLLM?

Câu trả lời ngắn nhất là:

> **Nếu mục đích chỉ là sử dụng model, chúng ta không cần ArcLLM.**

`llama.cpp` đã là một runtime rất trưởng thành. Mục tiêu công khai của dự án là thực hiện suy luận LLM bằng C/C++ với thiết lập tối thiểu và hiệu năng cao trên nhiều loại phần cứng. Qua thời gian, nó đã có rất nhiều backend, hỗ trợ CPU, nhiều dòng GPU và nhiều kiểu lượng tử hóa khác nhau.

Ollama lại giải quyết một nhu cầu ở lớp cao hơn: giúp người dùng tải, quản lý và chạy model thuận tiện hơn mà không cần tự xử lý tất cả chi tiết phía dưới.

Nếu mục tiêu là:

> “Tôi muốn tải một model về máy và chat với nó ngay.”

thì dùng một hệ thống trưởng thành như vậy hợp lý hơn rất nhiều so với tự viết runtime từ đầu.

ArcLLM không ra đời vì những công cụ ấy làm chưa tốt.

Ngược lại, chúng chính là một phần nguồn cảm hứng.

Một trong những điều rất thú vị từ Georgi Gerganov, `ggml` và sau đó là `llama.cpp` là việc họ cho thấy một hệ thống chạy model lớn không nhất thiết phải là một khối phần mềm bí ẩn chỉ tồn tại trong các framework khổng lồ.

Có thể đi xuống rất thấp.

Có thể nhìn thấy dữ liệu.

Có thể tự viết những phép tính.

Có thể tận dụng phần cứng phổ thông.

Và từ những thành phần tương đối nhỏ, từng bước hình thành một runtime thực sự hữu ích.

Tinh thần ấy là một trong những cảm hứng dẫn tới ArcLLM:

> **Muốn hiểu chiếc máy, hãy thử tự xây chiếc máy.**

Nhưng ArcLLM chọn không dùng llama.cpp làm lõi nghiên cứu.

Lý do không phải vì muốn “viết lại cho hay hơn”.

Nếu ngay từ đầu chúng ta lấy một runtime trưởng thành làm lõi, rất nhiều quyết định quan trọng đã được đưa ra hộ chúng ta: cách tổ chức tensor, cách chọn kernel, cách quản lý bộ nhớ, cách chia backend, cách điều phối công việc và hàng loạt tối ưu đã tích lũy qua nhiều năm.

Ta có thể đo chúng.

Ta có thể sửa một phần.

Nhưng sẽ khó hơn nhiều để trả lời một câu hỏi căn bản:

> **Tại sao kiến trúc lại phải có hình dạng như vậy?**

ArcLLM đi theo hướng ngược lại.

Bắt đầu với càng ít giả định càng tốt.

Khi cần đọc model, ta xây phần đọc model.

Khi cần đưa dữ liệu vào GPU, ta xây phần bộ nhớ GPU.

Khi cần một phép tính, ta xây phép tính đó.

Khi một cách tổ chức thất bại, ta giữ lại thất bại ấy.

Và chỉ khi bằng chứng cho thấy cần một abstraction mới — một lớp khái niệm mới — chúng ta mới thêm nó.

Chính vì vậy nhiều thứ mà ở runtime trưởng thành có thể chỉ xuất hiện như một API hoặc một cấu trúc dữ liệu, trong cuốn sách này sẽ có cả câu chuyện phía sau:

Tại sao nó xuất hiện?

Vấn đề nào buộc nó phải tồn tại?

Ta đã thử cách gì trước đó?

Cách nào thất bại?

Và bằng chứng nào khiến kiến trúc thay đổi?

## Vậy llama.cpp được dùng để làm gì?

Không dùng llama.cpp làm lõi không có nghĩa là bỏ qua nó.

Ngược lại, llama.cpp giữ một vai trò rất quan trọng trong ArcLLM:

> **Nó là một đối chứng bên ngoài.**

Hãy tưởng tượng ta tự chế một động cơ.

Hôm qua động cơ chạy được 10 km/h. Hôm nay sửa xong chạy 20 km/h.

Ta có thể vui vì nó nhanh gấp đôi.

Nhưng nếu những động cơ trưởng thành ngoài kia đã chạy 200 km/h thì “nhanh gấp đôi hôm qua” chưa nói được nhiều.

Nghiên cứu runtime cũng vậy.

ArcLLM có thể cải thiện 2 lần, 3 lần hay nhiều hơn so với chính phiên bản trước của nó. Những kết quả đó vẫn có giá trị vì chúng cho biết một thay đổi kiến trúc đã tạo ra tác động gì.

Nhưng chúng **không tự động chứng minh ArcLLM nhanh hơn một runtime trưởng thành**.

Muốn biết điều đó, cần đặt hai bên vào một phép so sánh công bằng: cùng model, cùng phần cứng, cùng đầu vào và cùng cách đo.

Đó là lý do llama.cpp xuất hiện nhiều lần trong câu chuyện ArcLLM.

Không phải như một đối thủ cần phải “đánh bại”.

Mà như một **thước đo thực tế**.

Một runtime nghiên cứu nếu chỉ tự so với chính mình rất dễ sống trong một thế giới riêng.

Đối chứng bên ngoài buộc chúng ta phải hỏi:

> Những gì vừa khám phá thực sự có giá trị đến đâu khi đặt cạnh một hệ thống đã trưởng thành?

Và như chúng ta sẽ thấy sau này, có lúc câu trả lời rất không dễ chịu.

Nhưng chính những lần như vậy lại mở ra những nhánh nghiên cứu quan trọng nhất.

## Bắt đầu từ đâu?

Bây giờ chúng ta đã biết ArcLLM nằm ở đâu.

Nó không phải toàn bộ ứng dụng AI.

Nó nằm ở lớp gần model và phần cứng.

Nó được sinh ra trên một chiếc máy có GPU Intel Arc 140V, lấy cảm hứng từ tinh thần xây runtime từ những thành phần thấp như ggml và llama.cpp, nhưng chọn tự xây đường thực thi của mình để mỗi quyết định kiến trúc đều có thể được quan sát và kiểm chứng.

Vậy bắt đầu xây một runtime như vậy từ đâu?

Có lẽ phản xạ đầu tiên là viết ngay một kernel thật nhanh.

ArcLLM không bắt đầu ở đó.

Trước khi tối ưu bất kỳ thứ gì, ta phải chắc rằng mình đang nhìn đúng thứ.

Giả sử hôm nay ta chạy một file model, ngày mai vô tình thay bằng file khác. Nếu kết quả nhanh hơn 10%, đó là nhờ runtime tốt hơn hay chỉ vì model đã thay đổi?

Hoặc hôm nay GPU có đủ bộ nhớ, ngày mai máy đang chạy nhiều chương trình khác và bộ nhớ trống giảm mạnh. Nếu lần chạy thứ hai thất bại, liệu kiến trúc có sai, hay chỉ đơn giản là chiếc máy đang bận?

Đó là lý do bước đầu tiên của ArcLLM có tên rất giản dị: **P0 — bootstrap runtime và khóa mục tiêu**.

Trong giai đoạn này, dự án chọn một model cụ thể để làm đối tượng nghiên cứu: Qwen2.5-Coder-1.5B.

Nhưng chỉ ghi tên model vẫn chưa đủ.

Hai file có thể có cùng tên mà nội dung khác nhau.

Vì vậy file được khóa bằng **SHA-256** — một dấu vân tay số. Chỉ cần nội dung bên trong thay đổi thì dấu vân tay cũng thay đổi.

P0 cũng kiểm tra xem máy có nhìn thấy GPU hay không, model có đọc được hay không và lượng bộ nhớ cần thiết có nằm trong khả năng phần cứng hay không.

Lần kiểm tra đầu tiên cho thấy file model chứa 338 tensor. Có thể tạm hiểu **tensor** là những bảng số mà model sử dụng trong các phép tính. Chúng ta sẽ mở chúng ra kỹ hơn ở chương sau.

Với ngữ cảnh 4.096 token, bộ lập kế hoạch ước lượng runtime cần khoảng 2,120 GiB bộ nhớ, trong khi ngưỡng khả năng đã được xác nhận là 3,75 GiB.

Không cần công thức phức tạp:

```text
Cần khoảng:      2,120 GiB
Có thể đáp ứng:  3,75 GiB

2,120 < 3,75
```

Về mặt năng lực, nó vừa.

Có một lần chạy bị chặn vì lượng RAM trống thực tế lúc đó thấp hơn mức dự phòng 8 GiB. Nhưng đó lại là một bài học quan trọng.

**“Máy đang thiếu bộ nhớ lúc này” không giống với “kiến trúc cần nhiều bộ nhớ hơn máy có thể cung cấp”.**

Một bên là trạng thái tạm thời.

Một bên là giới hạn của thiết kế.

Nếu không phân biệt hai chuyện đó, ta có thể giết một hướng nghiên cứu chỉ vì hôm ấy máy đang chạy quá nhiều chương trình.

Sau khi ranh giới này được làm rõ, P0 đạt PASS.

Và như vậy ArcLLM có viên gạch đầu tiên.

Chưa có model hoàn chỉnh đang trò chuyện.

Chưa có kernel tối ưu.

Chưa có những khái niệm phức tạp mà chúng ta sẽ gặp hàng chục chương sau.

Chỉ có một điều chắc chắn hơn trước:

> **Ta biết mình đang chạy model nào, file nào, trên phần cứng nào, và chiếc máy có đủ khả năng để bắt đầu hay không.**

Đó là một khởi đầu có vẻ nhỏ.

Nhưng mọi phép đo về sau đều phụ thuộc vào nó.

### Nhớ 3 điều

1. **ArcLLM là lớp runtime thực thi model, không phải toàn bộ hệ AI hay ứng dụng chat.**
2. **Tự xây runtime không phải vì llama.cpp hay Ollama không tốt, mà vì ta muốn nhìn thấy và kiểm chứng từng quyết định kiến trúc; llama.cpp vẫn là một đối chứng quan trọng.**
3. **Trước khi tối ưu, phải khóa đúng model, đúng file và đúng điều kiện phần cứng — nếu không, một con số đẹp hơn chưa chắc nói lên điều gì.**

**Chương 2 — Bên trong file model có gì?**

Chúng ta vừa biết P0 nhìn thấy 338 tensor trong một file GGUF.

Nhưng tensor là gì? Vì sao model lại chứa nhiều tensor như vậy? Và nếu model có hàng tỷ con số, runtime có thực sự phải bung tất cả chúng ra thành những con số lớn rồi mới tính được hay không?

Đó là nơi cuộc hành trình thực sự bắt đầu.
