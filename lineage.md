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
