# Project BLACKSWAN

## Enterprise Cyber Risk & GRC Assessment of the 2022 BNB Chain Bridge Exploit

*A post-incident security governance, risk assessment, control effectiveness and remediation engagement.*

| | |
|---|---|
| **Role** | Lead GRC Engineer |
| **Engagement type** | Independent cybersecurity risk assessment |
| **Subject incident** | BSC Token Hub cross-chain bridge exploit — 6 October 2022 |
| **Frameworks** | NIST CSF 2.0 · NIST SP 800-53 Rev. 5 · ISO/IEC 27001:2022 · COBIT 2019 · CIS Controls v8 · SOC 2 TSC (by analogy) |
| **Evidence basis** | Public sources only, individually cited (31 sources indexed) |

> **Scope discipline.** This assessment concerns the BSC Token Hub — the native bridge between BNB Beacon Chain and BNB Smart Chain, i.e. BNB Chain *protocol* infrastructure. It is **not** an assessment of Binance's centralised exchange; no exchange user funds were involved in the incident. Keeping that distinction straight is itself one of the findings (R-16).

---

## Why this project exists

Most breach write-ups explain what happened. This engagement demonstrates what a Senior GRC Engineer actually does with a breach: reconstruct it from evidence, identify why the control environment allowed it, quantify the enterprise risk, map remediation to recognised frameworks, and design the audit programme that would independently verify the fix. Every factual claim is cited; every figure that sources dispute is presented as an attributed range; and every judgement the public evidence cannot support is labelled an assessment limitation rather than dressed up as a finding.

## Headline facts (all cited in the report)

- On 6 October 2022 an attacker forged cross-chain proofs against the BSC Token Hub and caused the release of **2,000,000 BNB (two transactions of 1M) with no corresponding deposit**, exploiting a flaw in the Cosmos IAVL proof-verification code used by the bridge's `iavlMerkleProofValidate` precompile.
- The forged BNB was valued at **~$566M–$586M** depending on source and valuation moment — but that is *exposure created*, not loss. Estimates of value actually moved off-chain range from **$70M to $127M** by source and definition; stablecoin issuers froze **$33.5M**; the operator's settled figure is **~$100M unrecovered**. The report keeps these three numbers separate throughout.
- Detection was **public and community-driven** (on-chain analysts on Twitter) rather than demonstrably internal; validators halted the chain the same evening (~8 hours of downtime), and the bridge remained closed until the **Moran hard fork (12 Oct 2022)**. The durable redesign arrived with **BEP-171** (ICS23 verification, timelocks on massive transfers, automatic pause on forged-proof detection).
- **No public audit report scoped to the Token Hub bridge, its system contracts, or the proof-verification precompile could be located.** The report records this as an assurance *evidence gap* — existence cannot be established from public evidence — not as proof that no audit occurred.

## Deliverables

| Artefact | Location |
|---|---|
| Full investigation report (15 sections, ~22,600 words, architecture / attack-flow / attack-tree / IR decision diagrams) | [`report/REPORT.md`](report/REPORT.md) |
| Enterprise risk register — 16 risks, inherent & residual scoring (5×5), owners, KRIs | [`data/risk-register.csv`](data/risk-register.csv) |
| Control-gap assessment matrix — 15 domains, design & operating effectiveness, gap classification, test procedure per domain | [`data/control-gap-matrix.csv`](data/control-gap-matrix.csv) |
| Framework crosswalk — 35 mappings across NIST CSF 2.0, SP 800-53, ISO 27001:2022, COBIT 2019, CIS v8, SOC 2 | [`data/framework-crosswalk.csv`](data/framework-crosswalk.csv) |
| Remediation roadmap — 22 recommendations across 4 phases, each with owner, success criteria and evidence of completion | [`data/remediation-roadmap.csv`](data/remediation-roadmap.csv) |
| Internal audit & control testing plan — 7 fully specified tests, incl. replaying the historical forged proof as a regression case | [`audit/audit-test-plan.md`](audit/audit-test-plan.md) |
| Board-level risk briefing | [`board/board-briefing.md`](board/board-briefing.md) |
| Executive presentation (17 slides) | [`slides/Project-BLACKSWAN-Executive-Presentation.pptx`](slides/Project-BLACKSWAN-Executive-Presentation.pptx) |
| Assessment methodology, evidence hierarchy and limitations | [`methodology/methodology.md`](methodology/methodology.md) |
| Evidence index (31 sources), conflicting-figures register, "could not verify" list | [`evidence/evidence-and-sources.md`](evidence/evidence-and-sources.md) |

## The five questions the report answers

1. What happened, technically and operationally?
2. Why did the existing architecture and controls fail to prevent or contain it?
3. Which governance, risk and compliance weaknesses can *reasonably* be identified from evidence?
4. What should have been in place before the incident?
5. What corrective programme reduces the likelihood and impact of a recurrence — and how would an auditor independently verify it worked?

## Method in brief

Evidence was graded on a four-tier hierarchy (operator primary records and code releases → independent technical analyses → reputable reporting → unverified secondary claims, used only when labelled). Findings are classified as **confirmed**, **evidenced by observed response**, or **assessment limitations**. Risks are scored on a defined 5×5 matrix with residual ratings capped by stated discipline rules. Control gaps use a four-class taxonomy — *Absent / Ineffective / Insufficiently Evidenced / Cannot Be Established* — because "we found no public evidence of this control" and "this control failed" are different statements, and a credible assessor does not merge them. Full detail: [`methodology/methodology.md`](methodology/methodology.md).

## Limitations

This is a public-evidence assessment: no internal BNB Chain or Binance documents, personnel interviews, or system access were available. Where that limits a conclusion, the report says so. Nothing in this project asserts that any law or regulation was violated, and framework mappings are benchmarks for control design — not certification findings.

## Disclaimer

An independent portfolio assessment prepared for professional demonstration. Not affiliated with, commissioned by, or endorsed by Binance or BNB Chain. All incident facts are drawn from the cited public sources.

## License

MIT — see `LICENSE`.
