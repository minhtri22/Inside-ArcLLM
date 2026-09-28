# Chương 16 — Experiment 2×2: tách representation khỏi execution

> **Câu hỏi của chương:** Khi một phép tính đang chậm, làm thế nào biết nó chậm vì cách ta chia công việc cho GPU hay vì cách dữ liệu được tổ chức để GPU đọc?

Phần III kết thúc bằng một nguyên tắc quan trọng:

> **Sau một thay đổi lớn, phải đo lại hệ thống trước khi chọn thứ tiếp theo để tối ưu.**

Đó chính là điều ArcLLM làm.

Sau khi gate/up đã được tăng tốc mạnh, bản đồ bottleneck cũ không còn đủ an toàn để dùng như hiện tại.

Một vùng khác bắt đầu đáng chú ý: **FFN-down Q4_K**.

Trong model 7B đang xét, có 14 layer mà phép `FFN-down` dùng trọng số Q4_K với hình dạng:

```text
K    = 18944
rows = 3584
batch = 1
```

Ta đã biết một cách chia công việc mới — Split-K32 — có thể hữu ích.

Nhưng giờ xuất hiện một câu hỏi khác.

Có thể vấn đề không chỉ nằm ở:

> **cách tính**

mà còn nằm ở:

> **cách dữ liệu được bố trí để phục vụ phép tính đó.**

Đây là lúc ArcLLM phải tách hai khái niệm mà trước đó rất dễ bị trộn vào nhau.

## Execution và representation là hai chuyện khác nhau

**Execution — cách thực thi** trả lời:

> GPU chia công việc cho các đơn vị tính toán như thế nào?

Ví dụ đường cũ:

```text
1 invocation
→ 1 output row
→ tự quét toàn bộ K
```

Đường Split-K32:

```text
32 lane
→ cùng phụ trách 1 output row
→ chia chiều K cho nhau
→ gom kết quả
```

Đó là thay đổi **cách làm việc**.

Còn:

> **Representation — cách biểu diễn dữ liệu**

trả lời một câu khác:

> Các byte của trọng số được sắp xếp thế nào trong bộ nhớ để kernel đọc và sử dụng?

Hai representation có thể mang **cùng một nội dung toán học**, nhưng bố trí byte khác nhau.

Hãy tưởng tượng cùng một bộ hồ sơ.

Cách thứ nhất:

```text
tài liệu gốc
→ thông tin nằm rải ở nhiều mục
→ người xử lý phải đọc rồi giải mã
```

Cách thứ hai:

```text
vẫn đúng từng thông tin đó
→ nhưng trước khi làm việc đã sắp lại
→ những trường cần dùng được đặt gần và rõ hơn
```

Nội dung không đổi.

Nhưng chi phí lấy thông tin có thể đổi.

Đây là câu hỏi của Chương 16:

> **Nếu thay cách thực thi và thay cách biểu diễn đều làm phép tính nhanh hơn, làm thế nào biết lợi ích đến từ đâu?**

## Thay hai thứ cùng lúc sẽ làm mất câu trả lời

Giả sử ta làm một phiên bản mới:

```text
Serial-K cũ
+
Q4_K gốc
```

thành:

```text
Split-K32
+
representation mới
```

và candidate nhanh hơn 5×.

Ta biết gì?

Chỉ biết:

> hai thay đổi cộng lại tạo ra kết quả tốt hơn.

Ta không biết:

```text
Split-K đóng góp bao nhiêu?

representation đóng góp bao nhiêu?

cả hai có hỗ trợ nhau không?

hay thực ra chúng đang làm trùng một việc?
```

Đây là lý do cần một:

> **factorial experiment — thí nghiệm nhân tố**, ở đây là thiết kế **2×2**.

Không phải vì bảng 2×2 trông đẹp.

Mà vì nó cho phép tách hai biến.

## Bốn ô thay vì một candidate

Ta gọi hai yếu tố:

```text
A = thay cách chia công việc

B = thay representation
```

Ta có bốn trường hợp:

| | Representation gốc | Representation mới |
|---|---|---|
| **Serial-K** | **0** | **B** |
| **Split-K32** | **A** | **AB** |

Đọc từng ô:

### 0 — baseline

```text
Serial-K
+
Q4_K storage-native
```

`Storage-native — dạng lưu trữ gốc`, tức kernel đọc trực tiếp representation Q4_K vốn có trong model.

Đây là mốc để so.

### A — chỉ thay execution

```text
Split-K32
+
Q4_K gốc
```

Dữ liệu không đổi.

Chỉ thay:

> **cách chia chiều K cho GPU.**

Nếu A nhanh hơn 0, ta biết thay đổi execution có hiệu ứng.

### B — chỉ thay representation

```text
Serial-K
+
representation mới
```

Cách chia công việc vẫn Serial-K như baseline.

Chỉ thay cách dữ liệu được chuẩn bị cho execution.

Nếu B nhanh hơn 0, ta biết representation tự nó có hiệu ứng.

### AB — thay cả hai

```text
Split-K32
+
representation mới
```

Điểm rất quan trọng:

> **AB không được phép có optimization thứ ba.**

Nó phải đúng bằng:

```text
A
+
B
```

Nếu không, bảng 2×2 mất ý nghĩa.

## Representation mới vẫn phải là cùng dữ liệu

Representation B sau này được đặt tên là:

> **EXEC148**

Ta sẽ đi sâu hơn ở Chương 17.

Ở đây chỉ cần hiểu bản chất.

Một block Q4_K gốc chiếm:

```text
144 byte
```

Representation mới dùng:

```text
148 byte
```

Nhưng nó không được phép:

- đổi logical weight;
- bung toàn bộ sang FP16;
- bung toàn bộ sang F32;
- làm CPU model math ở mỗi token.

Nó chỉ:

> **sắp xếp lại thông tin Q4_K thành một hình thức thuận tiện hơn cho đường thực thi.**

Việc chuyển từ representation gốc sang representation mới được làm **một lần trước inference được đo**.

Sau đó GPU dùng image đã được chuẩn bị sẵn.

Đây gọi là:

> **materialization — tạo ra một biểu diễn thực thi cụ thể từ dữ liệu nguồn.**

Ta có thể hình dung:

```text
Q4_K gốc
↓
materialize một lần
↓
execution image
↓
GPU dùng lại cho nhiều token
```

Nếu phải materialize lại mỗi token thì đó là một bài toán hoàn toàn khác.

## Trước performance, bốn ô phải tính đúng

Một thí nghiệm 2×2 vô nghĩa nếu mỗi ô đang tính một thứ hơi khác nhau.

Vì vậy correctness phải được khóa trước.

Đặc biệt với representation mới, ArcLLM còn kiểm tra:

> khi giải mã representation gốc và representation mới về cùng một dạng logic, mọi giá trị Q4_K phải giống nhau.

Cả 14 tensor đều vượt kiểm tra này.

Sau đó ba candidate A, B và AB được so ở cấp phép tính.

Ngưỡng vẫn là:

```text
max_abs <= 0,02

RMSE <= 0,005
```

Kết quả A:

```text
max_abs
≈ 0,0000353

RMSE
≈ 0,00000368
```

AB có cùng mức sai lệch.

Lý do là cả A và AB dùng Split-K, nên thứ tự cộng FP32 thay đổi.

B thì thú vị hơn:

```text
max_abs = 0
RMSE    = 0
```

B vẫn giữ Serial-K và thứ tự cộng cũ.

Representation thay đổi nhưng kết quả phép tính vẫn bit-for-bit theo phép kiểm tra component này.

Quan trọng hơn, ở hai workload:

```text
W-S
W-C
```

cả bốn arm đều sinh đúng chuỗi 32 token đã khóa.

Tức:

```text
0
A
B
AB
```

đều đủ điều kiện bước sang timing.

## Timing cũng phải tránh lợi thế do thứ tự chạy

Nếu luôn chạy:

```text
0
A
B
AB
```

theo cùng một thứ tự, một arm có thể luôn gặp GPU lạnh hơn hoặc nóng hơn.

Vì vậy study dùng tám block với thứ tự được xoay.

Mỗi arm xuất hiện ở mỗi vị trí:

```text
thứ nhất
thứ hai
thứ ba
thứ tư
```

đúng hai lần.

Đây là một chi tiết nhỏ nhưng quan trọng.

Mục tiêu là làm cho câu hỏi:

> “A có nhanh hơn 0 không?”

ít bị trộn với:

> “A có tình cờ luôn chạy ở vị trí thuận lợi không?”

## Kết quả đầu tiên: cả A lẫn B đều có hiệu ứng

Ở workload W-S, median latency của họ Q4-down là:

```text
0   100,58 ms

A    38,37 ms

B    24,19 ms

AB   34,41 ms
```

So A với baseline:

```text
100,58 / 38,37
≈ 2,62×
```

Tức chỉ thay execution sang Split-K32 đã tạo speedup khoảng:

```text
2,62×
```

B:

```text
100,58 / 24,19
≈ 4,16×
```

Chỉ thay representation, vẫn giữ Serial-K, thậm chí tốt hơn A ở workload này.

Ở W-C:

```text
0   213,33 ms

A    37,84 ms

B    31,41 ms

AB   34,24 ms
```

Speedup của A:

```text
213,33 / 37,84
≈ 5,64×
```

B:

```text
213,33 / 31,41
≈ 6,79×
```

Như vậy có một kết luận khá mạnh:

> **Cả cách chia công việc và cách biểu diễn dữ liệu đều có thể tạo ra hiệu ứng latency lớn một cách độc lập.**

Nếu ta chỉ chạy AB ngay từ đầu, điều này đã bị che mất.

## Nhưng bất ngờ nằm ở ô AB

Trực giác rất dễ nói:

> “A tốt. B tốt. Vậy A+B phải tốt nhất.”

Nhưng W-S:

```text
B
24,19 ms

AB
34,41 ms
```

AB **chậm hơn B**.

W-C cũng vậy:

```text
B
31,41 ms

AB
34,24 ms
```

Một lần nữa:

> **kết hợp hai cơ chế tốt không tạo ra kết quả tốt nhất.**

Đây là lý do ô AB quan trọng.

Nó cho phép ta nhìn thấy:

> **interaction — sự tương tác giữa hai yếu tố.**

## Interaction nghĩa là gì?

Ta thử một ví dụ đời thường trước.

Baseline mất:

```text
100 giây
```

A riêng lẻ giảm còn:

```text
80 giây
```

tức A tiết kiệm:

```text
20 giây
```

B riêng lẻ giảm còn:

```text
70 giây
```

tức B tiết kiệm:

```text
30 giây
```

Nếu hai hiệu ứng cộng hoàn toàn độc lập, ta có thể kỳ vọng:

```text
100 - 20 - 30
= 50 giây
```

Nhưng AB thực tế lại:

```text
60 giây
```

Ta đã “mất”:

```text
10 giây
```

lợi ích so với kỳ vọng cộng đơn giản.

Một cách viết interaction là:

```text
G_INT
=
L_A + L_B - L_0 - L_AB
```

Thế ví dụ:

```text
80 + 70 - 100 - 60
=
-10
```

Interaction âm.

Ta gọi đây là:

> **antagonistic interaction — tương tác đối kháng**, tức hai thay đổi đang chồng lấn hoặc cản một phần lợi ích của nhau.

Trong experiment thật của ArcLLM, interaction cũng âm rõ rệt ở cả W-S và W-C.

Phân tích thống kê đã khóa trước cho kết luận:

> **ANTAGONISTIC_INTERACTION**

Nói đơn giản:

> **A tốt riêng. B tốt riêng. Nhưng hai lợi ích không cộng đẹp vào nhau.**

## Tại sao điều này lại quan trọng về kiến trúc?

Bởi nếu không có thí nghiệm 2×2, ta rất dễ xây một câu chuyện sai:

```text
Split-K tốt
+
representation mới tốt
=
hãy luôn dùng cả hai
```

Evidence nói khác.

AB nhanh hơn A một chút:

```text
W-S:
34,41 / 38,37
≈ 0,90

W-C:
34,24 / 37,84
≈ 0,91
```

tức representation mới vẫn giúp khi đặt trên Split-K.

Nhưng AB lại chậm hơn B:

```text
W-S:
34,41 / 24,19
≈ 1,42
```

và:

```text
W-C:
34,24 / 31,41
≈ 1,09
```

Tức khi representation mới đã xử lý một phần vấn đề, thêm Split-K không còn mang lại lợi ích như khi Split-K đứng một mình.

Hai cơ chế đang giải quyết những phần **không hoàn toàn độc lập** của chi phí.

Đây là tri thức kiến trúc mà một benchmark A/B đơn giản không thể cho ta.

## “Nhanh nhất” vẫn chưa chắc là kiến trúc nên chọn

Nhìn bảng timing, B nhanh nhất ở cả hai workload.

Có phải B mặc nhiên thắng?

Chưa.

B phải trả một cái giá.

Để có representation mới, runtime cần tạo thêm một execution image.

Trong study này, image đó chiếm thêm:

```text
549.527.552 byte
```

xấp xỉ:

```text
550 MB
```

bộ nhớ resident.

Ngoài ra còn có chi phí materialization một lần:

```text
≈ 231,64 ms
```

A thì khác.

A dùng representation Q4_K gốc.

Không cần image bổ sung.

Không có chi phí materialization đó.

Vì vậy ta có một trade-off:

```text
A
latency cao hơn B
nhưng
không tốn ~550 MB image mới
không tốn materialization ban đầu

B
latency thấp hơn
nhưng
có chi phí upfront + RAM
```

Bài toán kiến trúc vì vậy không còn là:

> “Ai có latency nhỏ nhất?”

Mà là:

> **“Với thời gian chạy và ngân sách bộ nhớ của ứng dụng thật, phương án nào có tổng chi phí phù hợp hơn?”**

## Bao nhiêu token thì B bắt đầu bù được chi phí ban đầu?

Ta có thể tính.

Với A:

```text
C_A(N)
=
N × L_A
```

Với B:

```text
C_B(N)
=
T_materialize
+
N × L_B
```

Trong đó:

- `N` là số token chịu chi phí của họ phép tính này;
- `L_A` là latency mỗi token của A;
- `L_B` là latency mỗi token của B;
- `T_materialize` là chi phí tạo representation B một lần.

Hai bên hòa nhau khi:

```text
N × L_A
=
T_materialize + N × L_B
```

Suy ra:

```text
N
=
T_materialize / (L_A - L_B)
```

Với W-S:

```text
T_materialize
≈ 231,64 ms

L_A
≈ 38,37 ms

L_B
≈ 24,19 ms
```

nên:

```text
N
≈ 231,64 / (38,37 - 24,19)

≈ 16,3 token
```

Theo mô hình chi phí đã khóa:

> **trước khoảng 16 token, A có lợi thế vì chưa phải trả materialization; sau đó B bắt đầu bù được chi phí thời gian ban đầu.**

Ở W-C:

```text
N
≈ 36,1 token
```

Nhưng nhớ rằng B vẫn cần thêm khoảng 550 MB resident memory.

Vì vậy crossover thời gian không tự động quyết định architecture.

## Không còn một “winner” duy nhất

Sau timing, hai arm bị loại khá rõ.

Baseline 0 bị A **dominate — lấn át**:

- cùng lớp chi phí không-materialization;
- nhưng A nhanh hơn.

AB bị B dominate:

- cùng phải có representation mới;
- cùng chịu lớp chi phí memory/materialization;
- nhưng B nhanh hơn ở cả hai workload.

Còn lại:

```text
A
vs
B
```

Không arm nào thắng tuyệt đối.

A tiết kiệm memory và startup cost.

B giảm steady-state latency hơn.

Hai phương án tạo thành:

> **non-dominated frontier — tập các phương án mà mỗi phương án còn một lợi thế riêng, nên không thể loại chỉ bằng một tiêu chí.**

Đây là một bước trưởng thành khác của cách nghĩ kiến trúc.

Không phải mọi experiment cuối cùng đều phải cho:

```text
WINNER = X
```

Đôi khi câu trả lời khoa học đúng là:

> **Có hai phương án hợp lệ cho hai chế độ sử dụng khác nhau.**

## Hardware counter cũng không được phép viết lại câu chuyện

Sau timing, ArcLLM thử dùng một số bộ đếm phần cứng để hiểu sâu hơn *vì sao* A và B có hiệu ứng.

Một số counter được kỳ vọng đo lượng dữ liệu đọc, mức sử dụng ALU và trạng thái stall.

Nhưng ba trong bốn counter cần thiết trả về:

```text
0
```

trong toàn bộ các quan sát.

Tức kênh đo đó không đủ thông tin để xác nhận đầy đủ mechanism.

Chỉ counter `SBID stall` cho tín hiệu có ích: A làm giảm rất mạnh loại stall này.

Điều phải làm lúc đó không phải:

> “Ba counter bằng 0, vậy mechanism không tồn tại.”

Mà là:

> **Kênh đo không đủ khả năng xác nhận mechanism ở mức chi tiết đó.**

Timing của A và B vẫn hợp lệ.

Interaction vẫn hợp lệ.

Chỉ có phần giải thích sâu bằng hardware counter là chưa được xác nhận đầy đủ.

Đây là một ranh giới rất quan trọng:

```text
effect được đo
≠
mọi chi tiết nguyên nhân đã được đo
```

## Thí nghiệm 2×2 đã thay đổi cách ta nhìn runtime

Trước experiment này, một optimization có thể được mô tả khá đơn giản:

> “Viết kernel nhanh hơn.”

Sau nó, câu chuyện trở nên khác.

Cùng một phép toán Q4-down có ít nhất hai trục tương đối độc lập:

```text
EXECUTION
GPU chia công việc thế nào?

REPRESENTATION
dữ liệu được chuẩn bị thế nào để execution đọc?
```

Và hai trục còn có thể tương tác.

Đây chính là lúc ta bắt đầu thấy một nhu cầu kiến trúc lớn hơn:

> **Có lẽ representation không nên chỉ là chi tiết ẩn bên trong một kernel.**

Nếu cùng một logical tensor có thể có:

```text
representation lưu trữ
```

và:

```text
representation phục vụ execution
```

thì runtime cần biết chúng khác nhau.

Nó cần biết:

- representation nào đang tồn tại;
- representation nào một executor cần;
- khi nào phải tạo nó;
- chi phí tạo bao nhiêu;
- nó sống bao lâu;
- và executor nào được phép dùng nó.

Những câu hỏi đó lớn hơn một shader Q4-down.

Đó là ranh giới dẫn tới Chương 17.

### Nhớ 3 điều

1. **Representation và execution là hai biến khác nhau.** Một cái quyết định dữ liệu được bố trí thế nào; cái kia quyết định GPU chia và thực hiện công việc thế nào. Thí nghiệm 2×2 cho phép thay từng yếu tố riêng rồi mới thử kết hợp.
2. **Hai optimization tốt riêng lẻ không nhất thiết cộng được với nhau.** A và B đều giảm latency mạnh, nhưng AB lại chậm hơn B ở cả hai workload; interaction được phân loại là đối kháng.
3. **Kiến trúc không chỉ được chọn bằng latency.** A không cần image phụ; B nhanh hơn nhưng cần materialization khoảng `231,64 ms` và thêm khoảng `550 MB` resident memory. Kết quả đúng có thể là một frontier, không phải một winner duy nhất.

**Chương 17 — EXEC148: khi bằng chứng buộc abstraction mới xuất hiện**

Experiment 2×2 vừa cho thấy một điều rất cụ thể:

> **Cùng một tensor logic có thể cần một cách biểu diễn khác khi bước vào execution — và cách biểu diễn đó có chi phí, vòng đời và giá trị riêng.**

Nếu vậy, representation không còn có thể được coi chỉ là vài byte layout nằm kín bên trong kernel.

Runtime phải bắt đầu hiểu nó như một đối tượng kiến trúc thực sự.
