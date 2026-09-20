# GUARDIAN 

A content-integrity verifier for LLM-generated outputs: detects when a hidden instruction embedded in an input (a document, email, webpage, or tool result) has silently steered an LLM's response, even when no privileged action or tool call occurred.

**Status: idea stage / pre-implementation.** This repo currently contains design docs and planning material, not a working system. Nothing here has been benchmarked. Treat every number in this README as a target, not a result.

---

## The problem, honestly

LLM agents read untrusted content — emails, uploaded documents, webpages, other agents' outputs — before responding. A hidden instruction in that content can quietly bias a summary, skew a recommendation, or alter a screening decision, without ever touching a tool call. Most agent-security work (CaMeL, Progent, FIDES, RTBAS, and similar) defends the *action* surface: it checks whether the agent is allowed to call a tool or take a step. None of it checks whether the agent's *words* were manipulated when no action was involved. That's the gap this project targets.

We are not the first to name this problem — "prompt-in-content attacks" is an existing term in the literature (2508.19287), and Microsoft's own 2026 security reporting documents it happening in production. **We have not found a deployed, evaluated detection system for it as of this writing.** That absence is the basis of the project, and it is also the biggest risk: if such a system exists and we simply haven't found it, the novelty claim weakens. This needs a proper literature search pass before Review 1, not just the searches done so far.

## Proposed approach

Generate the agent's response twice: once on the real (possibly poisoned) input, once on a sanitized/stripped version of the same input. Compare the two. A semantically significant divergence that can't be explained by legitimate content differences gets flagged.

**Known weaknesses of this approach, stated up front:**
- **Cost.** Two full generations per request roughly doubles inference cost and latency. This is a real deployment tax, not a footnote. Whether a cheaper proxy (e.g., from internal activations, without a second full generation) can approximate the same signal is an open question we have not started investigating — it's listed as future work because we don't yet know if it's feasible, not because we've deprioritized it.
- **Sanitization is not a solved step.** "Strip the hidden instruction to build the reference input" assumes we can reliably identify what to strip — which is close to solving detection already. Early versions will likely use a cruder sanitization (e.g., stripping all non-essential formatting/metadata, or using a trusted-source-only reconstruction) rather than true instruction removal. This needs to be designed carefully, and it may turn out to be the hardest part of the whole system.
- **Divergence ≠ attack.** Legitimate inputs can also cause large, valid differences in output (e.g., a document that legitimately changes the answer). Telling "the input legitimately changed the answer" apart from "a hidden instruction steered the answer" is the actual research problem, not a detail to sort out later.
- **No dataset exists yet that we've assembled.** The domains named (summarization, QA, resume screening) are a plan, not a built benchmark. Building or adapting one, with real prompt-in-content examples per domain, is unstarted work.

## Why this project (patent / paper / hackathon)

This repo exists to support a course's "Innovative Design Project" requirement, which is graded on patent acceptance, publication acceptance, or a hackathon win — not on the idea alone. That context matters for how this project is scoped:
- The system is framed as a deployable middleware layer (not a bare algorithm) to have a plausible path to patentability in India, where a bare ML method is excluded under Section 3(k). Whether an actual patent examiner agrees is unknown and won't be known for a long time.
- The generalization-evaluation angle (leave-one-dataset-out, following the 2026 "When Benchmarks Lie" methodology) is included specifically because it gives the project a rigorous, defensible evaluation story for a paper submission — not because it's the most exciting part of the system.
- Nothing here has been validated against a real external panel yet. The zeroth review is the first checkpoint.

## Roadmap (unstarted unless marked)

- [x] Literature scan of the gap (Track A / Track B / the open middle) — done, needs deepening
- [x] Title, abstract, zeroth-review slide deck — done
- [ ] Prototype: single-domain (summarization) dual-generation pipeline — not started
- [ ] Divergence scoring method (embedding distance vs. LLM-judge vs. something else — undecided)
- [ ] Sanitization strategy for the reference generation — undecided, likely the hardest open problem
- [ ] Dataset assembly across 3 domains — not started
- [ ] Leave-one-dataset-out evaluation harness — not started
- [ ] Overdefense / false-positive rate calibration on benign inputs — not started
- [ ] Cheap-proxy exploration (avoid double inference cost) — not started, feasibility unknown
- [ ] Patent draft — not started
- [ ] Paper draft — not started

## Open questions we don't have answers to yet

1. Is there existing work on output/content-integrity verification we've missed? (Needs a deeper search than what's been done so far.)
2. What counts as "semantically significant" divergence, precisely enough to threshold on?
3. Can sanitization be done reliably without essentially solving the detection problem first?
4. Does this generalize across genuinely different domains, or will it turn out to be summarization-specific?
5. What's the realistic false-positive rate on ordinary, benign content changes?

## Repo contents

- `docs/` — background research, gap analysis, zeroth-review slide deck (see `DomainShield_Zeroth_Review.pptx`)
- `src/` — empty; implementation has not started

## License

TBD.

## Contact

TBD — add team names, emails, and guide name here.
