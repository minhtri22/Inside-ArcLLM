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
