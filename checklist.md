# Checklist QA cuối — Inside ArcLLM

Ngày QA: **2026-09-28**

Trạng thái: **POST-PATCH QA CLOSED — 0 OPEN FINDINGS**

Tài liệu này ghi lại lần QA cuối cho toàn bộ nội dung đã xuất bản của **Inside ArcLLM — Xây dựng một runtime LLM từ những nguyên lý đầu tiên** gồm Lời nói đầu, Chương 1–20 và Bonus.

## Nguyên tắc QA

Nguồn được ưu tiên theo thứ tự:

1. final adjudication / canonical evidence artifact của experiment;
2. lineage.md append-only của ArcLLM hoặc nhánh nghiên cứu liên quan;
3. frozen spec / preregistration nếu chưa có adjudication cuối;
4. tài liệu nguồn chính thức bên ngoài khi nội dung không thuộc experiment ArcLLM.

Nếu spec sớm và adjudication cuối khác nhau, **adjudication cuối được ưu tiên**. Lỗi build, package, CI hoặc harness chỉ được coi là FAIL khoa học khi chính lineage/adjudication phân loại như vậy.

Bản checklist ban đầu được tạo ở chế độ **review-only**. Sau khi tác giả duyệt, **QA-01 đến QA-07 đã được vá đúng phạm vi và QA lại với cùng source hierarchy**. **QA-08 sau đó được mở lại và vá** để giữ đúng ranh giới xuất bản của phần Bonus.

## Quy ước

- **QA-CLEAN** — không phát hiện sai lệch cần vá trong phạm vi đã kiểm tra.
- **REVIEW** — có finding cần tác giả xem.
- **HIGH** — có thể làm người đọc hiểu sai nguyên nhân hoặc phạm vi bằng chứng.
- **MEDIUM** — claim/thuật ngữ cần thu hẹp hoặc làm rõ để khớp nguồn.
- **LOW** — số liệu đúng nhưng cách trình bày có thể gây hiểu nhầm, hoặc nên cập nhật bằng chứng mới hơn.

# Kết quả tổng quan

Đã QA **22 đơn vị nội dung**:

- 1 Lời nói đầu;
- 20 chương;
- 1 Bonus.

Kết quả sau author review + patch:

- **21 đơn vị QA-CLEAN**
- **Bonus đã được viết lại theo ranh giới xuất bản công khai**: chỉ giữ câu hỏi khái niệm chung, không chứa tên hay chi tiết nghiên cứu nội bộ;
- **0 finding còn mở**
- **QA-01 → QA-07: APPROVED_FIXED**
- **QA-08: APPROVED_FIXED — PUBLICATION-BOUNDARY CLEANUP**
- **Không phát hiện số liệu benchmark cốt lõi nào bị chép sai** trong các bảng/kết quả Q2, Q3, SA1, I002, I003 và Q4-down 2×2.

Các bản vá chỉ sửa **tên thuật ngữ, mechanism boundary, claim boundary, causal description và evidence provenance**. Không scientific verdict nào bị viết lại.

# Checklist từng phần

| Phần | Trạng thái | Kết quả QA |
|---|---|---|
| Lời nói đầu | QA-CLEAN | Không có số liệu khoa học cần đối chiếu; framing con người + AI không vượt claim nguồn. |
| Chương 1 | QA-CLEAN | P0: 338 tensor; ctx4096 ~2,120 GiB; frozen resident floor 3,75 GiB; 8 GiB launch reserve là advisory — khớp lineage. |
| Chương 2 | **QA-CLEAN** | **QA-01 APPROVED_FIXED:** GGUF được sửa thành `GGML Universal File`; số tensor và block size giữ nguyên. |
| Chương 3 | QA-CLEAN | P2: 980.097.536 byte, ~934,7 MiB, 4 arena <=256 MiB, scratch 64 MiB, 5 buffer sống — khớp lineage. |
| Chương 4 | QA-CLEAN | P3: glslang 16.5.0, 7 kernel gate, CPU reference độc lập — khớp lineage. |
| Chương 5 | QA-CLEAN | P4: 15 dispatch, 1 command buffer/submit/fence, no intermediate host read/write, numerical gate và infra-fail đều đúng. |
| Chương 6 | QA-CLEAN | P5: 28 layer, 338 tensor, 441 dispatch, top1=117612, final-hidden/logit error đúng. |
| Chương 7 | QA-CLEAN | P6: prompt [1,17,42,256], output [6228,17], RoPE base17, max_ctx16, K/V + logits error đúng. |
| Chương 8 | QA-CLEAN | P7-A/C/E/G/I/J/K/L/M/N/O và production-path closeout khớp lineage. |
| Chương 9 | QA-CLEAN | Q1/Q2 final evidence đúng; baseline llama.cpp dùng final corrected v0.4.1 commit b29c606e..., không dùng prelock cũ. |
| Chương 10 | QA-CLEAN | Q3: 40/40 fresh attempts, thresholds, 4 cell ratios và verdict FEASIBLE_NO_DEMONSTRATED_ADVANTAGE đúng. |
| Chương 11 | QA-CLEAN | 469 dispatch, 196 projection/GEMM, 84-dispatch fusion ceiling, 1,218× và 0,0276% đều khớp SA-H1/SA0. |
| Chương 12 | QA-CLEAN | Q4 aggregate ~3,137×/~3,148×, headroom/Amdahl và successor TTFT FAIL đều đúng. |
| Chương 13 | **QA-CLEAN** | **QA-02 APPROVED_FIXED:** làm rõ Q6 dùng candidate reader packed-Q6 riêng trong khi giữ cơ chế Split-K + geometry; verdict/timing boundary giữ nguyên. |
| Chương 14 | **QA-CLEAN** | **QA-02/03 APPROVED_FIXED:** Q6 transfer wording được thu hẹp; RMSE margin viết rõ `1 378×`; I002 T1/T2/T3 giữ nguyên. |
| Chương 15 | QA-CLEAN | I003: 20/20 pairs, 40/40 inferences, decode global ratio 10,3787×, E2E 9,9722×, throughput 0,09635 đúng. |
| Chương 16 | QA-CLEAN | 2×2 Q4-down: correctness, timing, antagonistic interaction, 231,6382 ms, 549.527.552 byte, crossover 16,33/36,07 đúng. |
| Chương 17 | **QA-CLEAN** | **QA-04 APPROVED_FIXED:** EXEC148 ghi chính xác `2 byte d + 2 byte dmin + 16 byte scale/min + 128 byte q`. |
| Chương 18 | **QA-CLEAN** | **QA-05 APPROVED_FIXED:** làm rõ P8 total capacity PASS; obstruction là inherited 256 MiB arena/single-tensor contract; P8 Phase2 chỉ là bounded mandatory-feasibility oracle. |
| Chương 19 | **QA-CLEAN** | **QA-06 APPROVED_FIXED:** mọi claim về mandatory P8 được bound về `bounded P8 oracle`; v4/114.688/Q4 backend result giữ nguyên. |
| Chương 20 | **QA-CLEAN** | **QA-07 APPROVED_FIXED:** thêm provenance rằng NPU numbers là analytical current-canonical projection + provider timing, không phải fresh full-model NPU benchmark. |
| Bonus | **QA-CLEAN** | **QA-08 APPROVED_FIXED:** viết lại theo ranh giới công khai; giữ hướng gợi mở chung, không công bố tên/kết quả/cơ chế của nghiên cứu nội bộ. |

# Findings cần tác giả duyệt

## QA-01 — APPROVED_FIXED — Chương 2 — tên đầy đủ của GGUF

**Mức:** MEDIUM  
**Loại:** factual terminology

### Nội dung hiện tại

> “GGUF là cách viết tắt thường dùng của GGML Universal Format”

### Đối chiếu nguồn

Tài liệu chính thức hiện tại của ggml-org/llama.cpp, file gguf-py/README.md, ghi:

> “GGUF (GGML Universal File) format.”

### Đề nghị vá

Đổi thành:

> **GGUF là viết tắt của GGML Universal File — một định dạng file nhị phân trong hệ sinh thái GGML.**

Không ảnh hưởng bất kỳ số liệu hay kết luận ArcLLM nào.

---

## QA-02 — APPROVED_FIXED — Chương 13 và mở đầu Chương 14 — “chuyển nguyên vẹn sang Q6”

**Mức:** MEDIUM  
**Loại:** mechanism/implementation precision

### Nội dung hiện tại

Chương 13 có cách diễn đạt:

> “Cùng cơ chế đã thắng ở Q4 có chuyển nguyên vẹn sang Q6 hay không?”

Chương 14:

> “...không giữ được tính đúng khi chuyển nguyên vẹn sang Q6_K.”

### Đối chiếu nguồn

Q6 **không dùng byte-identical Q4 shader**.

Q6 có một candidate shader riêng để đọc trực tiếp packed Q6_K. Thứ được giữ cố định từ Q4 là:

- subgroup32;
- workgroup128;
- 4 subgroup/workgroup;
- 1 subgroup/output row;
- K stride32;
- subgroup reduction;
- lane0 store;
- cùng ý tưởng Split-K / work decomposition.

Q6 final adjudication xác nhận compile/build PASS rồi candidate correctness FAIL; không có timing.

### Rủi ro

“Chuyển nguyên vẹn” có thể khiến người đọc hiểu rằng cùng một shader/binary được đem nguyên xi từ Q4 sang Q6.

### Đề nghị vá

Dùng:

> **“giữ nguyên cơ chế Split-K và hình học thực thi đã thắng ở Q4 khi chuyển sang một Q6_K candidate có bộ đọc packed-Q6 tương ứng”**

hoặc ngắn hơn:

> **“giữ nguyên cơ chế và hình học thực thi khi chuyển sang Q6_K.”**

Không thay verdict khoa học.

---

## QA-03 — APPROVED_FIXED — Chương 14 — cách viết khoảng cách RMSE 1.378×

**Mức:** LOW  
**Loại:** presentation ambiguity

### Nội dung hiện tại

~~~text
0,005 / 0,00000363
≈ 1.378
~~~

sau đó giải thích là “1.378 lần theo cách viết hàng nghìn của tiếng Việt”.

### Đối chiếu nguồn

Final T1 adjudication cho RMSE max:

~~~text
3,627874058914e-06
~~~

với gate:

~~~text
0,005
~~~

tức margin khoảng **1.378× theo nghĩa một nghìn ba trăm bảy mươi tám lần**, không phải 1,378 lần theo dấu thập phân kiểu Anh.

Con số nguồn là đúng; vấn đề chỉ là khả năng đọc nhầm.

### Đề nghị vá

Viết một trong hai:

> **≈ 1 378×**

hoặc:

> **≈ 1.378 lần — tức khoảng một nghìn ba trăm bảy mươi tám lần.**

---

## QA-04 — APPROVED_FIXED — Chương 17 — mô tả 4 byte đầu của EXEC148 còn quá gộp

**Mức:** LOW  
**Loại:** representation precision

### Nội dung hiện tại

~~~text
4 byte đầu
→ thông tin scale nền tảng

16 byte tiếp
→ các giá trị scale/min đã được đặt trực tiếp

128 byte còn lại
→ các giá trị Q4 được sắp lại theo thứ tự K
~~~

### Đối chiếu frozen EXEC148 design

Block 148 byte chính xác là:

~~~text
offset 0, size 2   → d raw FP16 bits
offset 2, size 2   → dmin raw FP16 bits
offset 4, size 16  → 8 cặp (scale,min) direct uint8
offset 20,size 128 → q values repacked theo increasing-K pairs
~~~

Mô tả hiện tại không sai về tổng kích thước, nhưng làm mất distinction d / dmin.

### Đề nghị vá

~~~text
2 byte đầu
→ d

2 byte tiếp
→ dmin

16 byte tiếp
→ 8 cặp scale/min trực tiếp

128 byte còn lại
→ q values được sắp lại theo thứ tự K
~~~

---

## QA-05 — APPROVED_FIXED — Chương 18 — P8 không FAIL vì thiếu tổng dung lượng bộ nhớ

**Mức:** HIGH  
**Loại:** causal / claim-boundary correction

### Nội dung hiện tại

> “Ở đây dữ liệu phải được chia thành một cách biểu diễn phân đoạn để phép tính có thể đi qua giới hạn bộ nhớ và tiếp tục chạy theo đường đã kiểm tra.”

### Đối chiếu P8 lineage

P8-A trên exact 7B target tính:

~~~text
total planned
= 5.347.770.372 byte

usable budget
= 16.374.562.816 byte

headroom
= 11.026.792.444 byte
~~~

**Tổng capacity PASS.**

P8-A FAIL vì một contract hẹp hơn:

> mỗi physical arena/tensor piece phải <=256 MiB.

Hai logical tensor vượt inherited single-tensor arena contract:

- token_embd.weight
- output.weight

P8-A2 sau đó dùng **row-aligned physical segmentation** để giải đúng obstruction này mà **không thay tổng memory formula, quantization, context hay KV precision**.

Trong Phase2 generic-acquisition study, P8 còn được dùng dưới dạng **bounded mandatory-feasibility oracle**; artifact Phase2 nói rõ P8-G vẫn FAIL và full inference không được promote trong claim đó.

### Rủi ro

Câu hiện tại dễ làm người đọc hiểu rằng 7B không fit tổng RAM/VRAM và segmentation là giải pháp cho thiếu capacity.

Đó không phải finding của P8-A.

### Đề nghị vá

Thay đoạn mở đầu họ thứ hai bằng ý chính:

> **“Ở một họ P8 có giới hạn, tổng dung lượng bộ nhớ thực ra đã PASS. Obstruction nằm ở contract mỗi physical arena không vượt 256 MiB: hai tensor vocab đơn lẻ lớn hơn giới hạn đó. Vì thế representation phân đoạn theo hàng trở thành điều kiện bắt buộc để đường thực thi bounded này khả thi mà không nới arena cap.”**

Sau đó nhắc rõ:

> **“Phase2 dùng trường hợp này như một bounded mandatory-feasibility oracle; nó không tự thân là claim full-inference P8.”**

Đây là finding quan trọng nhất của lần QA.

---

## QA-06 — APPROVED_FIXED — Chương 19 — cần gắn chữ “bounded” rõ hơn với họ P8

**Mức:** MEDIUM  
**Loại:** scope precision

### Nội dung hiện tại

Chương 19 nhiều lần gọi:

> “representation bắt buộc”

và kết luận:

> “Nó biểu diễn được họ representation bắt buộc.”

### Đối chiếu v2/v3/v4 artifacts

Final generic-surface evidence ghi rõ:

~~~text
P8 mandatory-feasibility bounded oracle
~~~

và:

~~~text
universality = false
proven_classes_only = true
future_counterexample_may_reopen = true
~~~

v2 adjudication còn ghi:

> “P8-G remains FAIL and full inference remains unauthorized.”

### Rủi ro

Phần cuối chương đã nói “tổng quát không có nghĩa phổ quát”, nên claim tổng thể đang đúng. Tuy nhiên một vài đoạn giữa chương có thể bị đọc như thể toàn bộ P8 production family đã được validation dưới abstraction này.

### Đề nghị vá

Ở lần giới thiệu họ thứ hai và ở “Nhớ 3 điều”, đổi thành:

> **“họ representation bắt buộc trong bounded P8 oracle đã kiểm tra”**

hoặc:

> **“bounded mandatory-feasibility P8 case.”**

Không cần đổi kiến trúc v4 hay số 114.688.

---

## QA-07 — APPROVED_FIXED — Chương 20 — số liệu NPU đúng nhưng cần gắn provenance “analytical projection”

**Mức:** MEDIUM  
**Loại:** evidence provenance

### Nội dung hiện tại

Chương 20 nêu:

- Gate/Up W-S ≈1,00×, W-C ≈1,04×;
- FFN-down warm transfer budget ≈66,66 / 24,20 ms/token;
- FP16 representation ≈3,54 GiB;
- naive 28-layer graph import ≈12,6 s;
- break-even ≈189 / 521 token.

### Đối chiếu final NPU artifacts

Các số trên đều đúng.

Nhưng source khóa rất rõ:

> current-canonical family shares là **analytical projection**, lấy exact-target post-I002 M1 evidence rồi áp Q4-down B/0 ratio và renormalize.

Artifact ghi:

> “This is an analytical current-canonical projection from exact-target evidence, not a fresh full-model timing measurement.”

Formal adjudication cũng ghi:

~~~text
fresh_full_model_run = false
NPU_backend_implemented = false
~~~

### Rủi ro

Chương đã nói NPU chưa tích hợp, nhưng người đọc có thể vẫn hiểu các con số trên là một full-model NPU benchmark trực tiếp của canonical runtime hiện tại.

### Đề nghị vá

Ngay trước các con số NPU thêm:

> **“Các con số sau là phép chiếu phân tích từ bằng chứng exact-target hiện có kết hợp với timing của NPU provider; đây không phải một fresh full-model benchmark có NPU.”**

Giữ nguyên tất cả giá trị số và conclusion: chỉ FFN-down được phép mở bounded study, không có production claim.

---

## QA-08 — APPROVED_FIXED — Bonus — ranh giới xuất bản công khai

**Mức:** HIGH  
**Loại:** publication boundary / confidentiality

### Finding

Bản Bonus cũ từng dùng trực tiếp tên và kết quả của một số chương trình nghiên cứu nội bộ để minh họa hướng quan sát hệ thống.

Các chi tiết đó không cần thiết cho mục tiêu của cuốn sách công khai và vượt quá ranh giới xuất bản mong muốn.

### Bản vá

Bonus đã được viết lại hoàn toàn theo nguyên tắc:

- chỉ giữ các khái niệm tổng quát: quan sát khác nguyên nhân, giới hạn của phép đo, sai khác nhỏ có thể đáng kiểm tra, trạng thái hệ thống thay đổi theo thời gian và can thiệp có kiểm soát;
- không nêu tên dự án nghiên cứu nội bộ;
- không nêu cây nghiên cứu, cơ chế riêng, kết quả PASS/FAIL hay thông số có thể dùng để suy ngược dự án nội bộ;
- giữ một cầu nối tự nhiên sang câu hỏi công khai: nếu đi theo một token xuyên qua cỗ máy thì ta sẽ thấy gì?

**Trạng thái: APPROVED_FIXED.**

Không verdict khoa học nào của ArcLLM bị thay đổi bởi bản vá này.

---

# Các số liệu trọng yếu đã đối chiếu và không phát hiện sai lệch

## Chương 1–8

- P0: 338 tensor; 2,120 GiB <= 3,75 GiB; launch reserve 8 GiB chỉ advisory.
- P1: F32=141, Q4_K=168, Q6_K=29, tổng 338.
- P2: 980.097.536 byte, 4 weight arenas, 64 MiB scratch.
- P4: 15 dispatch, final max_abs 0,000581026..., RMSE 0,0000302707....
- P5: 441 dispatch, top1 CPU/GPU 117612.
- P6: prompt [1,17,42,256], output [6228,17].
- P7-C: 5,182877989×.
- P7-E: 1,848765491×.
- P7-G: 1,224642504×.
- P7-I: 1,003404444× FAIL.
- P7-J: 0,9821774944× FAIL.
- P7-K: 1,068446103× FAIL.
- P7-L: 1,242926134× PASS.
- P7-N: 1,051612109× FAIL.
- P7-O: 0,8836691312× FAIL.
- P7-M: 441 prefill dispatch; 469 decode dispatch; fused gate/up 44,40335651%; FFN-down 30,42145996%; barrier/unattributed 0,0243244%.

## Chương 9–11

Q2 medians:

| Workload | System | TTFT ms | Decode tok/s | E2E ms |
|---|---:|---:|---:|---:|
| W-S | ArcLLM | 918,265 | 0,3184 | 98.285,231 |
| W-S | llama.cpp | 100,318 | 12,7449 | 2.535,868 |
| W-C | llama.cpp | 1.423,482 | 13,0354 | 3.803,029 |
| W-C | ArcLLM | 14.706,425 | 0,3990 | 91.043,729 |

Q3 fresh ratios:

| Cell | TTFT | Decode throughput | E2E | Working set |
|---|---:|---:|---:|---:|
| A/W-S | 12,422× | 0,02473× | 39,209× | 1,836× |
| A/W-C | 9,198× | 0,02172× | 31,355× | 1,829× |
| B/W-S | 14,801× | 0,01761× | 55,009× | 1,836× |
| B/W-C | 9,605× | 0,02800× | 26,451× | 1,829× |

SA-H1/SA0:
- 469 physical dispatch/token;
- 196 projection/GEMM dispatch/token;
- simple fusion ceiling 469/385 ≈1,218×;
- input reread proxy saving ≈0,0276% of 4,37 GB;
- Q3 decode gap roughly 35,7×–56,8×.

## Chương 12–15

SA1 Q4:
- aggregate A 3,1371499895×;
- aggregate B 3,1475012006×;
- min cells 1,659419× / 1,671474×.

Q6:
- compile/build PASS;
- candidate correctness FAIL;
- exact violated dimension unknown due fail-fast;
- timing pairs 0;
- target-model timing not run.

I002:
- real-weight component comparisons 72/72 PASS;
- max max_abs 3,814697265625e-05;
- max RMSE 3,627874058914e-06;
- decode geomean 2,19983×;
- E2E median-cell geomean 2,00263×;
- 20/20 measured pairs preserve token semantics;
- TTFT median ratios all <=1,10.

I003:
- 20/20 matched pairs; 40/40 inferences;
- global decode-latency ratio 10,378702×;
- global E2E-latency ratio 9,972199×;
- global throughput ratio 0,096351×.

## Chương 16–19

Q4-down 2×2 W-S medians:

~~~text
0  = 100,581013 ms
A  = 38,373384 ms
B  = 24,189140 ms
AB = 34,408567 ms
~~~

W-C:

~~~text
0  = 213,333823 ms
A  = 37,835155 ms
B  = 31,413020 ms
AB = 34,242629 ms
~~~

- interaction: ANTAGONISTIC_INTERACTION;
- EXEC148 materialization 231,6382 ms;
- incremental resident image 549.527.552 byte;
- A/B crossover W-S 16,3307 token, W-C 36,0687 token;
- 3/4 preregistered hardware counters structurally zero; mechanism attribution not confirmed through those channels;
- exact first-family policy preservation: 114.688 / 114.688;
- v4 closed surface: identity / execution_available / execution_ready / residency / acquisition / lifecycle;
- generic universality claim: false.

## Chương 20

Canonical runtime extraction:

- Profile0/Profile1 generated count: 32;
- prefill dispatches: 441;
- decode dispatches/step: 469;
- decode steps: 31;
- Q4 route B steps: 31;
- acquire/evict: 1/1;
- non-fixture input [1,42,314,2718];
- max new tokens 2;
- generated [2718,2718];
- request explicitly outside validated scientific domain;
- no new performance claim;
- no arbitrary-prompt quality claim;
- no persistent model session;
- no NPU integration;
- no fresh matched llama.cpp benchmark after canonical extraction.

NPU bounded-gate evidence:

- Gate/Up family ratio: 0,999666× W-S, 1,037576× W-C;
- FFN-down conservative remaining warm family budget: 66,66023 / 24,20116 ms/token;
- 28-layer FFN-down FP16 representation: 3,541015625 GiB;
- naive serial 28-layer import setup: 12.611,1776 ms;
- break-even: 189,186 / 521,098 token;
- production NPU value: not established;
- canonical NPU integration: not authorized.

# Nguồn QA chính

Các nguồn nghiên cứu nội bộ khác có thể được dùng ở phía tác giả để kiểm tra ranh giới phát biểu, nhưng **không được nêu tên, sao chép cơ chế hoặc tái xuất bản kết quả riêng trong repository công khai này**.

Không chép raw experimental evidence vào repository sách. Các nhóm nguồn đã dùng để đối chiếu:

- ArcLLM append-only lineage.md — P0–P8/Q1/Q2/Q3/P7.
- Q2 final adjudication.
- Q3 final adjudication.
- SA0 capability / SA-H1 decomposition.
- SA1 Q4 and Q6 final adjudications.
- I002 real-model component + final carry-through adjudications.
- I003 matched external final adjudication.
- Q4-down 4-arm correctness/timing/native-counter canonical artifacts.
- EXEC148 frozen layout design.
- Phase2 capability/acquisition v2, readiness v3, generic extension surface v4.
- Q4 Vulkan backend v4 revalidation and full-runtime integration QA.
- Canonical runtime extraction QA + post-commit QA.
- NPU capability/transfer Amdahl formal adjudication + analysis.
- Official ggml-org/llama.cpp gguf-py/README.md for the GGUF name expansion.

# Patch policy sau checklist

Kết quả sau khi tác giả duyệt:

~~~text
chapter patches applied = 7
bonus patches applied = 0
README content patches caused by QA = 0
scientific verdicts rewritten = 0
open findings = 0
~~~

Patch commits:

- Chương 2: `662933dcdbad8defd61f37ce432c1b8d97ebef6c`
- Chương 13: `db74e54df2227e35ce1a159ad30fcdde6bd58b37`
- Chương 14: `ce183c55d6d9726d81e1de81f7c0d10cbddf2d76`
- Chương 17: `67348852322ad71b549bc4c72813f9d441b62836`
- Chương 18: `6fed71848a9a19b6a286e645d3d95f54e025deb8`
- Chương 19: `1fb183dd35de93dbf7dc5d0014e52b73ee4599a3`
- Chương 20: `e8d538961512444740bdb882a6c659220101cbc3`

Post-patch QA xác nhận các đoạn đã sửa vẫn khớp bằng chứng khoa học cuối. Bonus sau đó được mở lại vì yêu cầu ranh giới xuất bản công khai và đã được viết lại mà không thay đổi verdict khoa học ArcLLM.


---

# PEDAGOGICAL QA — 2026-10-01

> **Nguồn kích hoạt QA:** phản hồi độc giả thực tế sau khi đọc hai chương đầu.
>
> **Phạm vi:** khả năng đọc của người bắt đầu từ số 0 về công nghệ/AI. Đây là QA sư phạm và ngôn ngữ, **không mở lại các verdict khoa học đã QA ở trên**.
>
> **Trạng thái:** **CLOSED — PEDAGOGICAL-QA-CLEAN AFTER FULL REWRITE**

## Tiêu chuẩn mới

Một đoạn chỉ được coi là đạt cho độc giả số 0 khi đồng thời thỏa:

1. **Tiếng Việt trước, thuật ngữ gốc sau.** Nếu có cách gọi tiếng Việt đủ đúng và dễ hiểu, dùng tiếng Việt làm câu chính; thuật ngữ tiếng Anh chỉ đặt trong ngoặc ở lần đầu cần tham chiếu.
2. **Không dùng trước khi dạy.** Một khái niệm không được xuất hiện như thể người đọc đã biết nó trước khi có một hình dung cơ bản.
3. **Dạy theo bậc thang.** Có thể bắt đầu bằng một mô hình gần đúng, nói rõ đây là cách hiểu tạm thời, rồi mới sửa dần tới bản chất chính xác hơn.
4. **Ví dụ đời thường trước sơ đồ kỹ thuật.** Sơ đồ chỉ củng cố một ý đã hiểu, không thay thế việc giải thích.
5. **Một đoạn chỉ nên đưa vào số ít khái niệm mới.** Không dồn một “từ điển thuật ngữ” vào đầu chương.
6. **Tên chương cũng phải đọc được.** Không được coi tiêu đề như vùng miễn trừ cho tiếng Anh chuyên môn.
7. **Tên riêng/chuẩn kỹ thuật được giữ nguyên khi cần** (ví dụ Vulkan, GGUF, llama.cpp, Q4_K), nhưng phải giải thích vai trò bằng tiếng Việt trước khi dựa vào tên đó.
8. **PASS/FAIL và các nhãn nghiên cứu phải có nghĩa tiếng Việt trước khi trở thành ký hiệu quen thuộc.**
9. **Độc giả không được buộc phải nhớ định nghĩa từ chương trước chỉ để hiểu câu hiện tại.** Khi một khái niệm quay lại sau khoảng cách dài, cần một lời nhắc tự nhiên nếu ngữ cảnh đòi hỏi.
10. **Bài kiểm tra cuối:** xóa các từ tiếng Anh không phải tên riêng khỏi đoạn văn; nếu ý chính trở nên khó hiểu hoặc mất nghĩa, đoạn đó chưa đủ Việt hóa.

## Kết quả tổng quan

**Kết luận ban đầu: FAIL — bản trước chưa đạt tuyên bố “bắt đầu từ số 0”. Sau vòng viết lại toàn sách, các finding PQA-01 → PQA-10 đã được vá và regression QA đóng sạch.**

Lý do không nằm ở độ sâu khoa học. Vấn đề chính là **cách dựng cầu tới độ sâu đó**.

Vòng sửa trước đã bổ sung Phần 0, glossary nội tuyến, mức đọc và sơ đồ xuyên suốt. Những thay đổi này có ích nhưng chưa giải quyết triệt để hai lỗi:

- câu văn vẫn trộn tiếng Việt và thuật ngữ Anh quá thường xuyên;
- sơ đồ/thuật ngữ xuất hiện trước khi người đọc có mô hình đời thường để bám vào.

## Findings

### PQA-01 — APPROVED_FIXED — HIGH — README tự mâu thuẫn với tuyên bố “không giả định đã biết AI”

README nói sách dành cho người bắt đầu từ số 0 nhưng ngay phần giới thiệu và mục lục dùng dày đặc:

- model;
- token;
- tensor;
- runtime;
- CPU/GPU;
- decoder layer;
- residency;
- production path;
- kernel;
- benchmark;
- correctness;
- representation;
- execution.

**Yêu cầu sửa:** README phải là phần dễ đọc nhất của repository. Dùng tiếng Việt làm chính; thuật ngữ gốc chỉ tham chiếu trong ngoặc khi cần.

---

### PQA-02 — APPROVED_FIXED — HIGH — Phần 0 đang hoạt động như “từ điển nén”, chưa phải cầu nhập môn

Ngay phần mở đầu đã yêu cầu người đọc nhìn đồng thời nhiều tầng:

`model → parameter/weight → dense → Transformer → token → tensor → CPU/GPU → runtime → quantization`.

Bản đồ hiện tại còn hiển thị trước các thuật ngữ như:

`parameters / weights`, `Decoder-only Transformer`, `RMSNorm / Attention / FFN`, `representation / lifecycle`.

Đây là tải nhận thức quá lớn cho người chưa có điểm tựa.

**Yêu cầu sửa:** Phần 0 phải đi từ một trải nghiệm quen thuộc — “gõ một câu, máy trả lời” — rồi mở từng hộp một. Không trình bày toàn bộ cây thuật ngữ trước.

---

### PQA-03 — APPROVED_FIXED — HIGH — Chương 1 chưa tạo được hình dung chắc chắn về “mô hình” và “hệ thực thi”

Cách giải thích hiện tại đúng về kỹ thuật nhưng vẫn trừu tượng:

> model chứa cấu trúc và hàng tỷ con số...
>
> runtime là hệ thực thi model...

Độc giả số 0 chưa có hình dung “một file mô hình nằm yên” khác “chương trình chạy mô hình” ở đâu.

**Yêu cầu sửa:** trước thuật ngữ phải có một ví dụ đời thường duy nhất, nhất quán. Ví dụ phải giúp phân biệt:

```text
thứ chứa những gì đã học
≠
thứ đọc và thực hiện nó
≠
phần cứng làm phép tính
```

Sau khi người đọc hiểu ba vai trò mới gắn nhãn:

`mô hình (model)`, `hệ thực thi (runtime)`, `bộ xử lý`.

---

### PQA-04 — APPROVED_FIXED — HIGH — Cần “định nghĩa bậc thang”, không cố chính xác tuyệt đối ngay câu đầu

Phản hồi độc giả về token chỉ đúng hướng ở phương pháp, không phải ở định nghĩa literal “mỗi từ cách nhau bằng dấu cách”.

Cách dạy phù hợp:

**Bậc 1 — đủ để đi tiếp**

> “Tạm hình dung token là một mảnh văn bản nhỏ; thường nó trông giống một từ hoặc một phần của từ.”

**Bậc 2 — sửa mô hình gần đúng**

> “Nó không nhất thiết là một từ. Bộ mã hóa của từng mô hình quyết định cách chia.”

**Bậc 3 — khi cần chính xác hơn**

> token ID, tokenizer vocabulary, khoảng trắng/dấu câu/subword...

**Yêu cầu sửa:** áp dụng cùng phương pháp cho tensor, layer, attention, cache, benchmark, quantization, representation và các khái niệm khó khác.

---

### PQA-05 — APPROVED_FIXED — HIGH — “Bản đồ xuyên suốt” hiện tại vi phạm luật không dùng trước khi dạy

Bản đồ hai cột được đặt ở đầu hầu hết chương và chứa cả các khái niệm của nhiều chương sau.

Với độc giả có nền, nó là định hướng.

Với độc giả số 0, nó trở thành một danh sách từ lạ lặp lại 20 lần.

**Yêu cầu sửa:** không tái sử dụng nguyên bản đồ đầy đủ ở đầu mọi chương.

Thay bằng **bản đồ mở dần**:

```text
Chương 1:
câu hỏi → mô hình → hệ thực thi → phần cứng

Chương 2:
câu hỏi → mô hình → [tệp mô hình / các khối số] → hệ thực thi → phần cứng

...

chỉ hiện thuật ngữ sau khi nó đã được dạy.
```

Nguyên tắc: bản đồ phải thể hiện kiến thức người đọc **đã có tới thời điểm đó**, không phải toàn bộ kiến thức tác giả đã biết.

---

### PQA-06 — APPROVED_FIXED — HIGH — Tiêu đề chương dùng tiếng Anh như thể người đọc đã biết

Các ví dụ nổi bật:

- “GGUF ... tensor store”
- “Full decoder residency”
- “KV cache”
- “Production path ... kernel”
- “Benchmark phải có đối chứng”
- “correctness”
- “Experiment 2×2: representation / execution”

**Yêu cầu sửa:** tiêu đề tiếng Việt trước. Thuật ngữ chuẩn có thể đặt sau trong ngoặc hoặc trong phần thân.

Ví dụ định hướng, chưa phải title final:

- “Bên trong tệp mô hình có gì? (GGUF)”
- “Giữ toàn bộ các lớp xử lý sẵn trong bộ nhớ”
- “Bộ nhớ giúp mô hình không phải tính lại từ đầu (KV cache)”
- “Đo tốc độ phải có một mốc để so sánh”
- “Nhanh nhưng sai thì vẫn là sai”
- “Tách cách sắp dữ liệu khỏi cách thực hiện phép tính”

---

### PQA-07 — APPROVED_FIXED — HIGH — Tần suất câu Việt–Anh trộn quá cao trên toàn sách

Audit từ Chương 1–20 cho thấy các từ `model`, `runtime`, `token`, `tensor`, `kernel`, `benchmark`, `decode`, `representation`, `execution`, `correctness`, `workload`, `baseline`, `speedup`... xuất hiện lặp lại dày đặc trong câu tiếng Việt.

Đây không còn là vấn đề “glossary lần đầu”; nó tạo cảm giác ngôn ngữ lai xuyên suốt.

**Yêu cầu sửa:** thiết lập canonical Vietnamese terminology và dùng nhất quán. Ví dụ:

- model → **mô hình**;
- runtime → **hệ thực thi**;
- token → giữ **token** sau khi đã định nghĩa vì không có một từ Việt thay thế đủ chính xác và phổ biến; trong giải thích dùng “mảnh văn bản” khi phù hợp;
- tensor → **khối số (tensor)** ở giai đoạn nhập môn, sau đó có thể dùng tensor khi người đọc đã quen;
- layer → **lớp**;
- decoder layer → **lớp giải mã**;
- benchmark → **phép đo so sánh / phép đối chứng hiệu năng** tùy ngữ cảnh;
- kernel → **chương trình tính toán nhỏ trên GPU (kernel)** rồi ưu tiên “phép tính GPU/chương trình GPU” trong văn xuôi;
- correctness → **tính đúng**;
- representation → **cách biểu diễn dữ liệu**;
- execution → **cách thực thi**;
- workload → **tải công việc / bài đo**;
- baseline → **mốc đối chứng**;
- speedup → **mức tăng tốc**;
- latency → **độ trễ**;
- throughput → **thông lượng / số token mỗi giây**, ưu tiên diễn giải bằng đại lượng cụ thể;
- memory → **bộ nhớ**;
- cache → **bộ nhớ đệm**.

Không áp dụng thay thế máy móc; câu phải được viết lại tự nhiên.

---

### PQA-08 — APPROVED_FIXED — MEDIUM/HIGH — PASS/FAIL đang đúng về nghiên cứu nhưng chưa thân thiện với độc giả nhập môn

PASS/FAIL là ngôn ngữ quản trị thí nghiệm của dự án và có giá trị lịch sử, nhưng xuất hiện dày có thể khiến sách giống báo cáo nghiên cứu.

**Yêu cầu sửa:** lần đầu và trong văn xuôi ưu tiên:

- **ĐẠT (PASS)**
- **KHÔNG ĐẠT (FAIL)**
- **CHƯA KẾT LUẬN ĐƯỢC (UNRESOLVED)**

Trong bảng/tóm tắt kỹ thuật có thể giữ ký hiệu PASS/FAIL sau khi người đọc đã quen.

---

### PQA-09 — APPROVED_FIXED — MEDIUM — Các phần nâng cao vẫn cần tiếng Việt, không được dùng nhãn “Nâng cao” để miễn giải thích

Chương 16–19 có mật độ rất cao của:

`representation`, `execution`, `residency`, `acquisition`, `lifecycle`, `creator`, `sidecar`, `critical path`, `abstraction`.

Đây là nơi dễ quay lại văn phong tài liệu kỹ thuật nhất.

**Yêu cầu sửa:** giữ độ sâu khoa học nhưng chuyển câu hỏi sang tiếng Việt:

```text
dữ liệu được biểu diễn thế nào?
ai tạo nó?
nó nằm ở đâu?
lấy nó bằng cách nào?
khi nào sẵn sàng?
giữ nó bao lâu?
```

Sau đó mới chỉ ra thuật ngữ gốc nếu nó giúp người đọc tra cứu.

---

### PQA-10 — APPROVED_FIXED — HIGH — Cần một “zero-reader regression test” cho mọi chương

QA hiện tại chủ yếu kiểm factual/scientific correctness. Cần thêm kiểm thử sư phạm.

Mỗi chương phải trả lời được:

1. Ba khái niệm mới đầu tiên là gì?
2. Chúng đã được giải thích **trước lần dùng có ý nghĩa đầu tiên** chưa?
3. Có câu nào bắt người đọc phải hiểu 3+ thuật ngữ mới cùng lúc không?
4. Có thể thay một từ Anh bằng tiếng Việt mà không mất nghĩa không?
5. Có ví dụ đời thường trước abstraction không?
6. Đoạn đầu tiên có khiến người đọc hiểu “chương này định giải quyết chuyện gì” mà không cần tra Google không?
7. “Nhớ 3 điều” cuối chương có viết bằng ngôn ngữ người mới có thể kể lại cho người khác không?

Chỉ khi cả 7 câu đều đạt mới coi chương là **PEDAGOGICAL-QA-CLEAN**.

### PQA-11 — APPROVED_FIXED — HIGH — Thuật ngữ gốc bị dịch mất trong ngoặc

Phản hồi độc giả phát hiện một lỗi sau vòng Việt hóa: một số khái niệm được viết kiểu:

`tín hiệu hoàn thành (tín hiệu hoàn thành)`

hoặc English-first kiểu:

`Driver — trình điều khiển`.

Cả hai đều không đạt mục tiêu của sách. Người đọc cần hiểu tiếng Việt ngay trong câu **và** cần thấy đúng thuật ngữ chuyên môn để nhận ra nó khi đọc tài liệu khác.

**Luật khóa:**

```text
Tiếng Việt (English)
```

Ví dụ:

- `tín hiệu hoàn thành (fence)`;
- `hàng đợi (queue)`;
- `vùng nhớ tạm (scratch)`;
- `chương trình GPU (kernel)`;
- `cơ chế chú ý (attention)`;
- `cách biểu diễn dữ liệu (representation)`;
- `trạng thái cư trú trong bộ nhớ (residency)`.

Đã quét lại Chương 0–20 + Bonus theo cả hai chiều:
- không để từ chuyên ngành tiếng Anh đứng trước rồi mới dịch;
- không dịch mất từ gốc bên trong ngoặc;
- tên riêng, mã kỹ thuật và định danh thật vẫn được giữ nguyên.

**Trạng thái: APPROVED_FIXED.**

---

## Kết quả sau vòng viết lại 2026-10-01

Toàn bộ phạm vi công khai đã được viết lại/QA:

- README;
- Lời nói đầu;
- Phần 0;
- Chương 1–20;
- Bonus;
- các sơ đồ giải thích;
- metadata biên tập công khai.

Regression QA xác nhận:

- tiêu đề chương dùng tiếng Việt dễ hiểu trước;
- các bản đồ đầu chương được mở dần theo kiến thức người đọc đã học, không còn lặp lại một “bức tường thuật ngữ” đầy đủ;
- `model/runtime/kernel/benchmark/correctness/representation/execution/workload/baseline/speedup...` không còn đứng trơ trong văn xuôi như kiến thức mặc định; từ gốc chỉ còn khi nằm trong ngoặc tham chiếu hoặc là định danh kỹ thuật cần giữ;
- các sơ đồ giải thích được Việt hóa; tên biến, tên chuẩn và mã kỹ thuật thật được giữ nguyên khi cần;
- `ĐẠT (PASS)` / `KHÔNG ĐẠT (FAIL)` được dùng theo hướng tiếng Việt trước;
- Bonus tuân thủ ranh giới xuất bản công khai;
- số liệu và scientific verdict của ArcLLM không bị viết lại.

**Pedagogical regression verdict: PASS — PEDAGOGICAL-QA-CLEAN.**

Điều này chỉ có nghĩa bản thảo đã vượt bộ tiêu chí QA sư phạm hiện tại. Phản hồi từ độc giả thật vẫn được ưu tiên để phát hiện những chỗ khó mà checklist không bắt được.

---

## Thứ tự sửa bắt buộc

Không sửa theo kiểu search/replace toàn sách.

Thứ tự phải là:

```text
README
↓
Lời nói đầu
↓
Phần 0
↓
Chương 1
↓
đọc thử như người số 0
↓
khóa giọng văn + bộ thuật ngữ tiếng Việt
↓
Chương 2–8
↓
QA hồi quy
↓
Chương 9–15
↓
QA hồi quy
↓
Chương 16–20 + Bonus
↓
full-book zero-reader QA
```

**Không được coi glossary là cách chữa cho một câu vốn đã khó hiểu.** Nếu câu chỉ hiểu được sau khi tra nghĩa của ba thuật ngữ, câu phải được viết lại.

