# Checklist QA cuối — Inside ArcLLM

Ngày QA: **2026-09-28**

Trạng thái: **REVIEW ONLY — CHƯA VÁ NỘI DUNG**

Tài liệu này ghi lại lần QA cuối cho toàn bộ nội dung đã xuất bản của **Inside ArcLLM — Xây dựng một runtime LLM từ những nguyên lý đầu tiên** gồm Lời nói đầu, Chương 1–20 và Bonus.

## Nguyên tắc QA

Nguồn được ưu tiên theo thứ tự:

1. final adjudication / canonical evidence artifact của experiment;
2. lineage.md append-only của ArcLLM hoặc nhánh nghiên cứu liên quan;
3. frozen spec / preregistration nếu chưa có adjudication cuối;
4. tài liệu nguồn chính thức bên ngoài khi nội dung không thuộc experiment ArcLLM.

Nếu spec sớm và adjudication cuối khác nhau, **adjudication cuối được ưu tiên**. Lỗi build, package, CI hoặc harness chỉ được coi là FAIL khoa học khi chính lineage/adjudication phân loại như vậy.

**Không có chương nào được sửa trong lần QA này.** Mọi mục bên dưới chỉ là finding để tác giả xem, duyệt rồi mới vá tài liệu.

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

Kết quả:

- **14 đơn vị QA-CLEAN**
- **8 đơn vị có REVIEW**
- **8 finding cần tác giả xem**
- **Không phát hiện số liệu benchmark cốt lõi nào bị chép sai** trong các bảng/kết quả Q2, Q3, SA1, I002, I003 và Q4-down 2×2.

Các finding còn lại chủ yếu thuộc bốn nhóm: **tên thuật ngữ**, **claim boundary**, **mô tả nguyên nhân**, và **bằng chứng mới hơn chưa được phản ánh**.

# Checklist từng phần

| Phần | Trạng thái | Kết quả QA |
|---|---|---|
| Lời nói đầu | QA-CLEAN | Không có số liệu khoa học cần đối chiếu; framing con người + AI không vượt claim nguồn. |
| Chương 1 | QA-CLEAN | P0: 338 tensor; ctx4096 ~2,120 GiB; frozen resident floor 3,75 GiB; 8 GiB launch reserve là advisory — khớp lineage. |
| Chương 2 | **REVIEW** | Số tensor và block size đúng; xem **QA-01** về tên đầy đủ GGUF. |
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
| Chương 13 | **REVIEW** | Q6 correctness FAIL đúng; timing=0 và target model not run đúng; xem **QA-02** về cụm “chuyển nguyên vẹn”. |
| Chương 14 | **REVIEW** | I002 số liệu T1/T2/T3 đúng; xem **QA-02** và **QA-03**. |
| Chương 15 | QA-CLEAN | I003: 20/20 pairs, 40/40 inferences, decode global ratio 10,3787×, E2E 9,9722×, throughput 0,09635 đúng. |
| Chương 16 | QA-CLEAN | 2×2 Q4-down: correctness, timing, antagonistic interaction, 231,6382 ms, 549.527.552 byte, crossover 16,33/36,07 đúng. |
| Chương 17 | **REVIEW** | EXEC148 và cost model đúng; xem **QA-04** về mô tả chính xác 4 byte đầu block148. |
| Chương 18 | **REVIEW** | Semantics acquisition/readiness đúng; xem **QA-05** vì mô tả P8 có thể làm sai nguyên nhân obstruction. |
| Chương 19 | **REVIEW** | v4 sáu chiều, 114.688 decisions và Q4 backend PASS đúng; xem **QA-06** về phạm vi bounded P8 oracle. |
| Chương 20 | **REVIEW** | Canonical runtime extraction đúng; NPU số liệu đúng; xem **QA-07** về provenance của các con số NPU. |
| Bonus | **REVIEW** | SIX root được kể đúng; xem **QA-08** vì latest SIX lineage đã đi xa hơn root branch. |

# Findings cần tác giả duyệt

## QA-01 — Chương 2 — tên đầy đủ của GGUF

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

## QA-02 — Chương 13 và mở đầu Chương 14 — “chuyển nguyên vẹn sang Q6”

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

## QA-03 — Chương 14 — cách viết khoảng cách RMSE 1.378×

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

## QA-04 — Chương 17 — mô tả 4 byte đầu của EXEC148 còn quá gộp

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

## QA-05 — Chương 18 — P8 không FAIL vì thiếu tổng dung lượng bộ nhớ

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

## QA-06 — Chương 19 — cần gắn chữ “bounded” rõ hơn với họ P8

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

## QA-07 — Chương 20 — số liệu NPU đúng nhưng cần gắn provenance “analytical projection”

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

## QA-08 — Bonus — SIX root đúng nhưng chưa phản ánh các replication mới hơn

**Mức:** MEDIUM  
**Loại:** latest-lineage completeness, không phải contradiction

### Nội dung hiện tại

Bonus kể đúng kết luận của nhánh SIX gốc:

- không hỗ trợ intrinsic-frequency interpretation;
- không thấy abrupt mode switch trong miền đã thử;
- strong context spectral-modulation claim FAIL;
- native geometry phần lớn sụp sau matched intervention interface;
- complete local tangent field được xác nhận trong nhánh gốc;
- không có token→physical-electrical claim.

Các câu đó **không sai**.

### Nhưng latest SIX lineage đã đi tiếp

#### SIX_R1 — structural replication

Formal close:

> **BOUNDED_TANGENT_MECHANISM_REPLICATED**

Cơ chế tangent-field được replicate qua ít nhất hai họ smooth recurrent dynamics cấu trúc khác nhau ở perturbation hữu hạn |epsilon|=0.25.

#### SIX_R2 — non-smooth switching

Formal close:

> **PASS WITH BOUNDARY REFINEMENT**

Khi crossing switching surface, frozen baseline tangent trở nên stale; một **endogenous branch-aware piecewise response law** tái tạo response trong tested system.

#### SIX_R3 — explicit second-order dynamics

Formal close:

> **AUGMENTED_MARKOV_STATE_REQUIRED_AND_SUFFICIENT**

Current 16-D state FAIL; full augmented 32-D Markov state PASS. Explicit two-lag tangent numerically equivalent với augmented-state tangent, không phải causal law bổ sung.

### Đề nghị vá

Không cần biến Bonus thành một chương SIX dài.

Có thể thêm một box ngắn sau phần root SIX:

> **“Sau nhánh SIX gốc, cơ chế này tiếp tục bị thử phá.”**

Rồi tóm tắt R1/R2/R3 trong ba đoạn ngắn và giữ nguyên non-claims:

- chưa universal;
- chưa physical hardware dynamics;
- chưa token-to-electrical encoding.

Nếu tác giả muốn Bonus chỉ kể đúng thời điểm lịch sử của root SIX thì có thể **không vá**, nhưng nên thêm một câu xác định mốc thời gian để tránh người đọc hiểu đây là trạng thái nghiên cứu mới nhất.

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
- Token X-Ray current README for its public capability boundary.
- SIX root scientific report + append-only lineage.
- SIX_R1 / R2 / R3 formal branch closures.
- Official ggml-org/llama.cpp gguf-py/README.md for the GGUF name expansion.

# Patch policy sau checklist

Tại thời điểm tạo checklist:

~~~text
chapter patches applied = 0
README content patches caused by QA = 0
scientific verdicts rewritten = 0
~~~

Đề nghị khi tác giả duyệt:

1. duyệt từng finding hoặc duyệt theo nhóm;
2. vá đúng finding đã duyệt, không rewrite chương ngoài phạm vi;
3. QA lại riêng những đoạn đã vá với cùng source hierarchy;
4. cập nhật checklist finding thành APPROVED_FIXED hoặc REJECTED_NO_CHANGE, giữ lịch sử thay vì xóa finding.
