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
