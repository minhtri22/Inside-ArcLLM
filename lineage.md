# Public editorial lineage

> Append-only editorial history for the public book repository. This file records publication/editorial state only. It does not reproduce the private scientific lineage or runtime artifacts.

## 2026-09-28 — Public repository initialized

Repository:
`minhtri22/Inside-ArcLLM`

Purpose:
- publish only author-approved book material;
- keep the ArcLLM research/source repository private;
- prevent runtime source code, raw artifacts, private branches, internal research records, or implementation-only material from leaking into the book repository.

Publication boundary:

```text
private research source-of-truth
        ↓ editorial review
author-approved book content
        ↓
Inside-ArcLLM public repository
```

Initial public set:
- Lời nói đầu — AUTHOR APPROVED;
- Chương 1 — AUTHOR APPROVED v0.3;
- Chương 2 — AUTHOR APPROVED with GGUF-expansion condition applied;
- Chương 3 — AUTHOR APPROVED with contract/term-reminder conditions applied;
- Chương 4 — AUTHOR APPROVED;
- Chương 5 — AUTHOR APPROVED with governance/runtime separation condition applied.

Not published at initialization:
- Chapter 6 draft;
- runtime source code;
- raw scientific artifacts;
- private scientific lineage;
- private QA/runtime packages;
- research branches or commit history from the private repository.

## Editorial rules carried into the public book

1. Start from zero; no coding prerequisite.
2. Technical English terms receive a short Vietnamese meaning reminder when needed to preserve reading flow.
3. Scientific PASS/FAIL, implementation/execution, governance, and infrastructure must not be conflated.
4. A governance artifact such as `lineage.md` must be labeled as governance and not implied to participate in runtime execution.
5. Research-management modes are introduced only when the chronology actually requires them.
6. New chapters are published only after explicit author approval.

## 2026-09-28 — Terminology refinement: intermediate host round-trip

Chapter 5 now introduces the reader-facing term:

`intermediate host round-trip`
→ vòng lặp tính toán trung gian quay ngược về CPU.

It is explicitly connected to the P4 contract phrase `zero intermediate host read/write`.

Editorial consequence:
- Chapter 5 introduces and explains the term.
- Chapter 6 (P5) reuses it as already-known terminology.
- Chapter 7 (P6) uses the term directly without re-explaining it.


## 2026-09-28 — Chapter 6 AUTHOR APPROVED

Approved:
`chapters/06-full-decoder-residency.md`

Applied before approval:
- token-selection wording narrowed to the P5 question: CPU/GPU agreement on `top1`; sampling policy is outside this chapter's scope;
- `intermediate host round-trip` is reused from Chapter 5 terminology rather than reintroduced from scratch;
- generation bridge uses the Vietnamese-first phrase `vòng lặp tạo sinh tự hồi quy (autoregressive generation)`.

Scientific scope remains P5:
- 338/338 tensors and 980,097,536 packed bytes resident;
- 28 decoder layers + final RMSNorm + tied Q6_K LM head;
- 441 Vulkan dispatches in one command buffer / one submit / one fence wait;
- no intermediate host round-trip;
- final-normalized-hidden and logits correctness gates PASS;
- CPU/GPU `top1 = 117612`;
- no performance or multi-token-generation claim.


## 2026-09-28 — Chapter 7 AUTHOR APPROVED

Approved:
`chapters/07-kv-cache-model-bat-dau-nho-token-truoc.md`

Scientific scope remains P6:
- four-token prefill prompt `[1, 17, 42, 256]`;
- persistent GPU-resident K/V across all 28 layers;
- no intermediate host round-trip for K/V;
- CPU orchestration reads logits/top1 but does not read/write K/V;
- independent CPU and GPU KV states;
- CPU/GPU greedy outputs `[6228, 17]` agree exactly;
- prefill logits, decode logits, K cache and V cache gates PASS;
- no performance, long-sequence or production-path claim.

Editorial boundary:
- `intermediate host round-trip` is reused without re-explaining the term;
- sampling remains outside scope; greedy argmax is used only as the fixed correctness rule;
- Chapter 8 may now draft P7 production-path optimization and evidence-driven stopping.


## 2026-09-28 — Chapter 8 AUTHOR APPROVED

Approved:
`chapters/08-production-path-khong-den-tu-mot-kernel-than-ky.md`

Scientific scope remains P7:
- P7-A scales the proven path to pp512/tg128 and identifies device-side execution as the first evidenced bottleneck class;
- P7-B/D/H/M re-profile the production graph rather than optimizing blindly;
- P7-C, P7-E, P7-G and P7-L are correctness-preserving optimization PASSes under their own frozen contracts;
- P7-I/J/K/N/O remain first-class negative evidence;
- P7-L is the frozen production-path winner, not a claim of global optimality or superiority to llama.cpp;
- cross-run absolute throughput drift is not used to override same-run interleaved A/B evidence.

Terminology refinement:
- `family` is introduced as **họ tác vụ tính toán (compute/kernel family)**: tasks related by role, primitive or execution mechanism, not an arbitrary grouping of unrelated computations.
- Chapter 8 introduces research modes E/M/C/T as a reader-facing governance framework for later work and explicitly does not retroactively relabel historical P7 execution.


## 2026-09-28 — Chapter 9 AUTHOR APPROVED

Approved:
`chapters/09-benchmark-phai-co-doi-chung.md`

Required editorial corrections applied before approval:
- replaced the unclear phrase `mảnh chronology` with reader-facing Vietnamese: `một bước đã xảy ra trước đó trong hành trình nghiên cứu`;
- introduced `p95` before use as the 95th percentile, with a concrete 100-measurement interpretation and the reason five samples per cell are insufficient for a stable tail metric.

Scientific scope remains Q2 matched characterization:
- exact same 7B GGUF bytes on ArcLLM and pinned llama.cpp v0.4.1 Vulkan baseline;
- raw token IDs, context 4096, F32 KV, greedy generation and fixed 32-token output;
- W-S prompt-4 and W-C prompt-256 workloads;
- one warmup plus five measured attempts per system/workload cell;
- TTFT, decode throughput, E2E latency and resource characterization;
- 20/20 measured attempts succeeded;
- formal Q2 classification is `Q2_MATCHED_CHARACTERIZATION_COMPLETE`;
- Q2 remains characterization-only and does not itself adjudicate a winner or regime advantage.


## 2026-09-28 — Chapter 10 AUTHOR APPROVED

Approved:
`chapters/10-khi-tu-build-duoc-van-chua-du.md`

Scientific scope remains Q3 no-practical-advantage confirmation:
- unchanged ArcLLM architecture, same model/hardware/baseline and the frozen W-S/W-C workloads;
- two fresh independent sessions, four cells per session, one warmup + five measured attempts per cell;
- 40/40 fresh measured attempts succeeded;
- primary benefit dimensions are TTFT, decode throughput, E2E latency and peak working set;
- preregistered practical-effect thresholds and blocking-harm guard remain fixed;
- no primary benefit passes in either session/workload and blocking-harm fails throughout;
- formal verdict is `FEASIBLE_NO_DEMONSTRATED_ADVANTAGE`, not `UNRESOLVED`;
- current ArcLLM architecture line closes under the preregistered stop rule.

Editorial boundary:
- negative result is framed as a bounded result for the frozen model/hardware/workloads/architecture, not a universal claim about ArcLLM or llama.cpp;
- successor work may only reopen through a separate mechanism-grounded architecture intervention review.


## 2026-09-28 — Chapter 11 AUTHOR APPROVED WITH CONDITIONS APPLIED

Approved:
`chapters/11-tu-that-bai-sang-mot-cau-hoi-dung-hon.md`

Author conditions applied before publication:
- removed the meta-editorial sentence `Không nên bắt người đọc phải tự dịch cụm này.`; the text now moves directly from the technical phrase to its Vietnamese decomposition;
- introduced **kernel fusion — gộp kernel** before repeated use, with a concrete two-kernel-to-one-kernel example and its intended effects;
- introduced **primitive — thao tác nền tảng** at SA0-CAP before reuse, so the hardware-capability discussion does not depend on unexplained English terminology.

Scientific/editorial scope:
- Chapter 11 closes Part II;
- the current Q3 architecture remains closed;
- `Successor Architecture (SA)`, `SA-H1`, `SA0` and `SA0-CAP` are introduced as bounded research terms, not as proof of a successful new architecture;
- successor hypothesis formation uses ArcLLM internal evidence plus published literature as hypothesis support only;
- no private external research project is named or imported into the public book narrative;
- SA0/SA0-CAP establish causal/capability qualification only, not successor performance;
- the chapter explicitly distinguishes same-hardware llama.cpp evidence from causal attribution: the matched comparison proves the hardware/model pair can run much faster than current ArcLLM decode, but does not by itself prove GEMM is the entire root cause.

## 2026-09-28 — 20-chapter / 4-part outline locked

The public book outline is now frozen at 20 main chapters across four parts:

1. **Part I — Build the Machine (Ch. 1–8)**  
   Goal: build a real runtime from model data, Vulkan and primitive computation through decoder, KV cache and the evidence-selected production path.

2. **Part II — Để evidence phán xét (Ch. 9–11)**  
   Goal: place the runtime under matched comparison, accept the Q3 negative verdict, close the current architecture, and define the conditions for opening a successor hypothesis. Part II ends at Chapter 11.

3. **Part III — Kiến trúc chỉ có giá trị khi đi qua thực tế (Ch. 12–15)**  
   Goal: compress multiple architecture hypotheses into their scientific lessons; show that component gains must survive correctness, transfer and end-to-end constraints; weave practical human+AI research governance modes E/M/C/T into the chronology rather than giving them a detached management chapter.

4. **Part IV — Từ runtime cụ thể tới abstraction tổng quát (Ch. 16–20)**  
   Goal: derive representation/execution, acquisition/lifecycle and generic runtime abstractions only when evidence requires them, then close the volume with validated boundaries and remaining open questions.

Additional outline decisions:
- Ledger64 is not a standalone public-book chapter; historical evidence remains in lineage/source-of-truth but the main narrative stays compact;
- former Ch. 12/15/16 material is compressed into the new Part III arc, with the value converging at the whole-system/Amdahl lesson;
- the correctness/Q6 story remains an independent Chapter 13;
- Token-XRay no longer has a main chapter; it appears only lightly as an internally created observability tool, with optional deeper treatment in a bonus section;
- a research experiment does not automatically become a chapter. A chapter must introduce a concept, change belief, close a path, or force an architectural boundary.


## 2026-09-28 — Chapter 11 FINAL AUTHOR APPROVAL / README reader QA

Chapter 11 final author approval confirmed after all conditional edits were applied.

Public README QA:
- rewritten as a reader-facing landing page rather than an author/editorial planning note;
- separates currently published reading links (Foreword + Chapters 1–11) from the future roadmap;
- preserves the approved 20-chapter / 4-part structure and each part's goal;
- removes author-facing commentary about why Token-XRay does or does not receive a main chapter;
- future sections are labeled as upcoming content rather than as already published chapters;
- editorial principles are phrased as promises to the reader rather than internal workflow instructions.


## 2026-09-28 — Chapter 12 AUTHOR APPROVED

Approved:
`chapters/12-nhieu-kien-truc-nhung-chi-thuc-te-moi-tra-loi-duoc.md`

Editorial/scientific scope:
- Chapter 12 opens Part III;
- E/M/C/T is briefly reintroduced near the start so readers do not need to return to Chapter 8 after a four-chapter gap;
- the chapter then focuses on E (Explore) and M (Mechanism): AI may expand the idea space, but only one sufficiently clear mechanism should consume new confirmatory evidence;
- subgroup32 Split-K Q4 evidence is presented as component evidence only, not as an ArcLLM-wide speed claim;
- Amdahl/headroom is introduced to separate local speedup from potential whole-system value;
- a historical integrated-successor negative is used only for the lesson that decode/E2E gains can coexist with a blocking TTFT regression.

## 2026-09-28 — Chapter 13 AUTHOR APPROVED WITH CONDITION APPLIED

Approved:
`chapters/13-khi-correctness-noi-khong.md`

Author condition applied before publication:
- changed the reader-facing heading from the metaphorical `Q4 PASS không có hộ chiếu sang Q6` to `Q4 PASS không có nghĩa là Q6 cũng vậy`.

Scientific/editorial scope:
- Chapter 13 centers Mode C (Confirm);
- the unchanged subgroup32 Split-K mechanism that passed Q4_K is tested against Q6_K under frozen correctness gates;
- Q6 shader compile/build succeeds, but candidate-vs-CPU correctness fails before performance measurement;
- Q6 timing remains explicitly unknown: zero measured pairs, no target-model run, and no performance claim;
- the chapter preserves the distinction between implementation/infrastructure success, scientific correctness, and performance;
- the valid Q6 correctness FAIL is preserved as a generalization boundary rather than rescued by threshold or geometry changes.


## 2026-09-28 — Chapter 14 AUTHOR APPROVED

Approved:
`chapters/14-tu-mot-co-che-tot-toi-he-thong-that.md`

Scientific/editorial scope:
- Chapter 14 centers Mode T (Transfer / carry-through);
- the already validated Q4 subgroup32 Split-K mechanism is transferred without broadening scope: decode only, Q4_K only, gate/up only, 56 substitutions per token;
- T1 real-model transfer preserves correctness across 72/72 comparisons using real weights and baseline-produced activations;
- T2 preserves full-model 32-token semantics in all required warmup and measured pairs;
- T3 shows material internal ArcLLM carry-through: global decode geomean approximately 2.20× and median-cell E2E geomean approximately 2.00×, with TTFT non-regression gate passing;
- the chapter explicitly distinguishes internal ArcLLM carry-through from an external llama.cpp advantage claim, which requires a fresh matched comparison for the new candidate;
- human+AI governance emphasis: AI executes the bounded transfer work while human authority preserves scope, gates, and claim boundaries.


## 2026-09-28 — Chapter 15 AUTHOR APPROVED

Approved:
`chapters/15-mot-kien-truc-chi-thang-khi-toan-he-duoc-loi.md`

Scientific/editorial scope:
- Chapter 15 closes Part III;
- the closed I002 candidate is re-benchmarked against the pinned llama.cpp Vulkan baseline using a fresh paired matched characterization rather than historical Q2 numbers;
- 20/20 matched pairs and 40/40 measured inferences are valid;
- the fresh post-I002 gap remains large: global cell-median geomean decode-latency ratio approximately 10.38× and E2E-latency ratio approximately 9.97× candidate/llama;
- historical Q2 and fresh I003 results are not algebraically combined into a causal gap-closure estimate;
- the chapter emphasizes that a large successful intervention can invalidate the old bottleneck ranking, so measurement must restart before selecting another mechanism;
- Part III closes with E/M/C/T explicitly framed as a loop: Transfer changes the system, then measurement reopens Explore;
- human+AI governance converges on preserving question scope, claim boundaries, stale-evidence awareness, and the authority to stop or re-measure rather than optimizing reflexively.


## 2026-09-28 — Chapter 16 AUTHOR APPROVED

Approved:
`chapters/16-experiment-2x2-tach-representation-khoi-execution.md`

Scientific/editorial scope:
- Chapter 16 opens Part IV;
- the Q4-down causal study is presented as a 2×2 factorial separation between execution/work decomposition (A) and execution representation (B);
- all four arms pass the frozen correctness gate before timing;
- A and B independently reduce Q4-down latency, while AB is slower than B in both workloads and the preregistered interaction is antagonistic;
- architecture cost is included: B requires one-time materialization and an additional resident execution image, creating A-vs-B crossover rather than a universal winner;
- the chapter introduces the non-dominated frontier concept and preserves the boundary that partial/inadequate hardware counters cannot overwrite valid timing evidence;
- the conclusion is architectural: representation and execution are distinct axes, motivating a runtime-level representation abstraction rather than another isolated kernel tweak.


## 2026-09-28 — Chapter 17 AUTHOR APPROVED WITH VIETNAMESE TERMINOLOGY CONDITION APPLIED

Approved:
`chapters/17-exec148-khi-bang-chung-buoc-mot-lop-truu-tuong-moi-xuat-hien.md`

Author condition applied before publication:
- performed a full Vietnamese-first terminology pass rather than patching only the cited sentences;
- removed mixed constructions such as `Một possibility...`, `lúc inference bắt đầu`, and `FAIL và trade-off...`;
- changed the chapter title from `abstraction` to `lớp trừu tượng`, retaining English only as a lookup term at first explanation;
- `execution`, `representation`, `layout`, `primitive`, `creator`, `residency`, `tuple`, `critical path`, `sidecar`, and related specialist terms are now introduced in Vietnamese first and then used primarily in Vietnamese;
- reader-facing prose now prefers `cách thực thi`, `cách biểu diễn dữ liệu`, `bố cục dữ liệu`, `khối chức năng nền tảng`, `nơi tạo`, `nơi cư trú`, `vòng đời`, `đường chuyển giao`, and `chính sách duy trì`;
- the final transition to Chapter 18 is likewise rewritten in Vietnamese rather than relying on bare `residency/acquisition/lifecycle` terminology.

Scientific/editorial scope:
- EXEC148 remains an execution representation of the same logical Q4_K data, not a new model or quantization;
- exact logical tuple equality across all 14 target tensors is preserved as the representation-equivalence basis;
- A remains the validated no-extra-representation Split-K32 primitive and B remains the validated EXEC148 representation primitive with unresolved placement/lifetime policy;
- B's measured latency value, one-time materialization cost, and approximately 550 MB incremental resident image motivate runtime-level representation identity, placement, reuse, and lifetime questions;
- the abstraction is presented as evidence-driven: the 2×2 study and cost frontier create the need for it rather than a top-down desire for generic architecture.


## 2026-09-28 — Chapter 18 AUTHOR APPROVED WITH VIETNAMESE TERMINOLOGY QA

Approved:
`chapters/18-du-lieu-o-trong-bo-nho-van-chua-du.md`

Author condition applied before publication:
- performed a full Vietnamese-first terminology pass across the chapter;
- replaced mixed prose such as `resident = true?`, `request`, `fallback`, `policy`, `session`, `image`, `backend`, `state machine`, and similar half-English constructions with Vietnamese explanations first;
- English terms are retained only as lookup handles where useful: residency, acquisition, lifecycle, execution readiness/availability, evict, backend, memory pressure;
- the chapter title and reader-facing transitions are fully Vietnamese.

Scientific/editorial scope:
- residency is narrowed to presence of separately acquired representation only;
- acquisition is split between optional reuse-amortized acquisition and mandatory-for-feasibility acquisition;
- NOT_READY and OUTSIDE_VALIDATED_CAPABILITY are kept as distinct no-route outcomes;
- the direct Split-K32 holdout demonstrates that execution readiness cannot be encoded by representation residency;
- execution availability, execution readiness, residency, acquisition and lifecycle are separated as independent questions;
- lifecycle preserves expensive valid representations independently of temporary execution-readiness loss;
- a concrete Q4 Vulkan backend is cited only as evidence that these distinctions can be obeyed without hidden family-specific semantics;
- no new performance claim is introduced.


## 2026-09-28 — Chapter 19 AUTHOR APPROVED

Approved:
`chapters/19-tu-arcllm-cu-the-toi-mo-hinh-runtime-tong-quat-hon.md`

Scientific/editorial scope:
- Chapter 19 consolidates the evidence-driven runtime surface into six independent dimensions: identity, execution availability, execution readiness, representation residency, acquisition, and lifecycle;
- the surface is tested across three distinct demonstrated classes: optional reuse-amortized represented execution, mandatory feasibility-enabling representation, and direct execution requiring no extra representation;
- all 114,688 first-family policy decisions remain preserved after the abstraction changes;
- one boolean execution-ready state remains sufficient for the tested transition/adversarial states; no larger activation state machine is justified by current evidence;
- fallback readiness is explicit rather than assumed;
- a concrete demand-driven Q4 Vulkan backend revalidates the six-way distinction without hidden eager acquisition or family-specific policy semantics;
- the historical 4-arm experimental harness is explicitly distinguished from a production lifecycle-bound backend after a prior binding FAIL;
- v4 is framed as bounded-general, not universal, and introduces no new performance claim.


## 2026-09-28 — Chapter 20 AUTHOR APPROVED; MAIN BOOK COMPLETE

Approved:
`chapters/20-ta-da-hieu-runtime-den-dau.md`

Author condition applied before publication:
- the book name is now presented Vietnamese-first, English-second:
  - `Inside ArcLLM — Xây dựng một runtime LLM từ những nguyên lý đầu tiên`
  - `Building an LLM Runtime from First Principles`
- all “Tập 1” framing was removed from Chapter 20 and the public roadmap;
- Chapter 20 closes Inside ArcLLM as a complete standalone book rather than promising a second volume;
- the closing transition points only to an optional Bonus about observing the running system.

Scientific/editorial scope:
- canonical runtime extraction is presented as architecture convergence, not new science or new performance evidence;
- frozen W-S/W-C regression behavior, dispatch topology and lifecycle behavior remain preserved after extraction;
- a non-fixture request demonstrates reusable request-level runtime behavior but is explicitly not a language-quality validation;
- prior internal carry-through and fresh external characterization remain separate claims;
- generic-v4 architecture PASS is kept separate from performance claims;
- NPU remains outside the canonical runtime: only bounded FFN-down transfer study authorization exists, with long-lived/cached-session constraints and no production-speedup claim;
- PASS and FAIL lineage is retained as part of the book's core epistemic result.

Publication state:
- Foreword + Chapters 1–20 are AUTHOR APPROVED and public;
- the 20-chapter main book is complete;
- future Bonus/Epilogue material remains optional and does not imply a second volume.


## 2026-09-28 — Bonus AUTHOR APPROVED; README ROADMAP COMPLETED

Approved:
`chapters/bonus-tu-xay-co-may-toi-lang-nghe-co-may.md`

Bonus editorial scope:
- opens from the completed ArcLLM runtime story toward broader ways AI might help observe and reason about complex systems;
- Token X-Ray is presented as an evidence-linking/orchestration layer rather than a generic profiler replacement;
- SIX is presented as a controlled numerical perturbation program, not as biological EEG, physical electrical token encoding, or a claim of a new universal theory;
- the bonus preserves major SIX FAIL/UNRESOLVED results, including unsupported intrinsic-frequency, strong mode-switch, fixed-geometry, and strong context-modulation interpretations;
- the strongest retained SIX claim remains local and bounded: under small perturbations and a common intervention interface, causal-response geometry is explained by an operating-point-conditioned complete local tangent field;
- future applications to factories, robotics, infrastructure, digital twins and other complex systems are explicitly framed as research directions, not demonstrated outcomes;
- the book remains standalone and does not promise a second volume.

README roadmap QA:
- Part I now mirrors Parts III–IV with direct chapter links and one-sentence summaries for Chapters 1–8;
- Part II now mirrors Parts III–IV with direct chapter links and one-sentence summaries for Chapters 9–11;
- the Bonus is linked from both the main reading list and the supplemental-material section;
- publication status now records Foreword + Chapters 1–20 + Bonus as public.


## 2026-09-28 — FINAL EVIDENCE QA PATCH SET APPROVED AND CLOSED

Author decision:
- patch every evidence-QA finding affecting the main book;
- leave the Bonus unchanged because it is intentionally an open research direction rather than a current-state SIX survey.

Applied evidence-alignment patches:
- Chapter 2: corrected GGUF expansion to **GGML Universal File**;
- Chapter 13: clarified that Q6 preserves the Q4 Split-K mechanism/geometry but uses a Q6-specific packed reader rather than a byte-identical Q4 shader;
- Chapter 14: applied the same Q6 transfer boundary and removed ambiguity in the ~1,378x RMSE margin by writing **1 378x**;
- Chapter 17: made the EXEC148 block layout exact: 2-byte d, 2-byte dmin, 16-byte direct scale/min pairs, 128-byte repacked q payload;
- Chapter 18: corrected the P8 causal description — total memory capacity passed; the obstruction was the inherited <=256 MiB single-tensor/arena contract, resolved by row-aligned physical segmentation without changing quantization/context/KV precision or total-memory formula; Phase2 use is explicitly bounded mandatory-feasibility evidence, not a new full-inference claim;
- Chapter 19: bounded mandatory-representation claims to the tested P8 oracle and retained the explicit non-universality boundary;
- Chapter 20: made NPU provenance explicit — the reported current-canonical NPU numbers are analytical projections from exact-target evidence plus provider timing, not a fresh full-model NPU benchmark and not canonical NPU integration.

Patch commits:
- Chapter 2: `662933dcdbad8defd61f37ce432c1b8d97ebef6c`
- Chapter 13: `db74e54df2227e35ce1a159ad30fcdde6bd58b37`
- Chapter 14: `ce183c55d6d9726d81e1de81f7c0d10cbddf2d76`
- Chapter 17: `67348852322ad71b549bc4c72813f9d441b62836`
- Chapter 18: `6fed71848a9a19b6a286e645d3d95f54e025deb8`
- Chapter 19: `1fb183dd35de93dbf7dc5d0014e52b73ee4599a3`
- Chapter 20: `e8d538961512444740bdb882a6c659220101cbc3`

Bonus disposition:
- QA-08 = **ACCEPTED_NO_PATCH**;
- the root SIX narrative remains scientifically valid for the direction-opening purpose of the Bonus;
- later SIX_R1/R2/R3 results remain outside this Bonus scope and are not backfilled.

Final QA state:
- 21 content units QA-CLEAN;
- 1 Bonus ACCEPTED_NO_PATCH;
- 0 open findings;
- no benchmark number or scientific verdict was rewritten;
- no README change was required.
