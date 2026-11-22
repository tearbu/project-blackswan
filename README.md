# Project BLACKSWAN

## Enterprise Cyber Risk & GRC Assessment of the 2022 BNB Chain Bridge Exploit

*A post-incident security governance, risk assessment, control effectiveness and remediation engagement.*

| | |
|---|---|
| **Role** | Lead GRC Engineer |
| **Engagement type** | Independent cybersecurity risk assessment |
| **Subject incident** | BSC Token Hub cross-chain bridge exploit, 6 October 2022 |
| **Frameworks** | NIST CSF 2.0 · NIST SP 800-53 Rev. 5 · ISO/IEC 27001:2022 · COBIT 2019 · CIS Controls v8 · SOC 2 TSC (by analogy) |
| **Evidence basis** | Public sources only, individually cited (31 sources indexed) |

> **Scope discipline.** This assessment is about the BSC Token Hub, the native bridge between BNB Beacon Chain and BNB Smart Chain. That is BNB Chain *protocol* infrastructure. It is **not** an assessment of Binance's centralised exchange, and no exchange user funds were involved in the incident. Keeping that distinction straight is itself one of the findings (R-16).

---

## Why this project exists

Most breach write-ups stop at what happened. This one tries to show what a Senior GRC Engineer does next. I reconstructed the incident from the evidence, looked at why the control environment allowed it, quantified the enterprise risk, mapped remediation to recognised frameworks, and designed the audit programme that would check independently whether the fix worked. Every factual claim is cited. Where sources disagree on a figure, the report gives the attributed range instead of picking one. And where the public evidence cannot support a judgement, it says so and labels it an assessment limitation rather than dressing it up as a finding.

## Headline facts (all cited in the report)

- On 6 October 2022 an attacker forged cross-chain proofs against the BSC Token Hub and caused the release of **2,000,000 BNB (two transactions of 1M) with no corresponding deposit**. The flaw was in the Cosmos IAVL proof-verification code used by the bridge's `iavlMerkleProofValidate` precompile.
- The forged BNB was valued at **~$566M to $586M** depending on source and valuation moment, but that is *exposure created*, not loss. Estimates of value actually moved off-chain range from **$70M to $127M** by source and definition. Stablecoin issuers froze **$33.5M**. The operator's settled figure is **~$100M unrecovered**. The report keeps these three numbers separate throughout.
- Detection was **public and community-driven** (on-chain analysts on Twitter) rather than demonstrably internal. Validators halted the chain the same evening (~8 hours of downtime), and the bridge stayed closed until the **Moran hard fork (12 Oct 2022)**. The durable redesign arrived with **BEP-171** (ICS23 verification, timelocks on massive transfers, automatic pause on forged-proof detection).
- **No public audit report scoped to the Token Hub bridge, its system contracts, or the proof-verification precompile could be located.** The report records this as an assurance *evidence gap*. In other words, existence cannot be established from public evidence. That is not proof that no audit occurred.

## Deliverables

| Artefact | Location |
|---|---|
| Full investigation report (15 sections, ~22,600 words, architecture / attack-flow / attack-tree / IR decision diagrams) | [`report/REPORT.md`](report/REPORT.md) |
| Enterprise risk register: 16 risks, inherent & residual scoring (5×5), owners, KRIs | [`data/risk-register.csv`](data/risk-register.csv) |
| Control-gap assessment matrix: 15 domains, design & operating effectiveness, gap classification, test procedure per domain | [`data/control-gap-matrix.csv`](data/control-gap-matrix.csv) |
| Framework crosswalk: 35 mappings across NIST CSF 2.0, SP 800-53, ISO 27001:2022, COBIT 2019, CIS v8, SOC 2 | [`data/framework-crosswalk.csv`](data/framework-crosswalk.csv) |
| Remediation roadmap: 22 recommendations across 4 phases, each with owner, success criteria and evidence of completion | [`data/remediation-roadmap.csv`](data/remediation-roadmap.csv) |
| Internal audit & control testing plan: 7 fully specified tests, incl. replaying the historical forged proof as a regression case | [`audit/audit-test-plan.md`](audit/audit-test-plan.md) |
| Board-level risk briefing | [`board/board-briefing.md`](board/board-briefing.md) |
| Executive presentation (17 slides) | [`slides/Project-BLACKSWAN-Executive-Presentation.pptx`](slides/Project-BLACKSWAN-Executive-Presentation.pptx) |
| Assessment methodology, evidence hierarchy and limitations | [`methodology/methodology.md`](methodology/methodology.md) |
| Evidence index (31 sources), conflicting-figures register, "could not verify" list | [`evidence/evidence-and-sources.md`](evidence/evidence-and-sources.md) |

## The five questions the report answers

1. What happened, technically and operationally?
2. Why did the existing architecture and controls fail to prevent or contain it?
3. Which governance, risk and compliance weaknesses can *reasonably* be identified from evidence?
4. What should have been in place before the incident?
5. What corrective programme reduces the likelihood and impact of a recurrence, and how would an auditor independently verify it worked?

## Method in brief

Evidence was graded on a four-tier hierarchy: operator primary records and code releases first, then independent technical analyses, then reputable reporting, then unverified secondary claims (used only when labelled). Findings are classified as **confirmed**, **evidenced by observed response**, or **assessment limitations**. Risks are scored on a defined 5×5 matrix, with residual ratings capped by stated discipline rules. Control gaps use a four-class taxonomy: *Absent / Ineffective / Insufficiently Evidenced / Cannot Be Established*. That matters because "we found no public evidence of this control" and "this control failed" are different statements. A credible assessor does not merge them. Full detail: [`methodology/methodology.md`](methodology/methodology.md).

## How the assessment was built

The work ran in a fixed order. First, the incident was reconstructed from cited public sources: operator statements and code releases, independent technical analyses, and reputable reporting, each graded on the four-tier hierarchy described above. Second, the reconstruction was turned into the working artefacts in `data\`: a risk register scored on the 5×5 matrix, a control-gap matrix across 15 domains, and a crosswalk mapping each gap to the frameworks. Third, the gaps drove a phased remediation roadmap, and the roadmap drove an audit test plan designed to verify the fixes independently, including replaying the historical forged proof as a regression case. The narrative report explains the reasoning, the board briefing compresses it, and the CSVs hold the underlying analysis in a form a reader can sort, filter, and challenge.

### Decisions

- **Public sources only, every claim cited.** A portfolio assessment has to be checkable by any reader, so the evidence base is public and the index names all 31 sources. The cost is real: with no internal documents, interviews, or system access, some questions can only be answered "cannot be established", and the report says so instead of filling the gap.
- **Conflicting figures kept as attributed ranges.** The forged BNB is often reported as one headline number, but exposure created, value moved off-chain, and value unrecovered are three different measures. The report keeps them separate throughout rather than picking the largest figure.
- **"No public evidence of a control" is not scored as "the control failed".** The gap taxonomy has separate classes for Absent, Ineffective, Insufficiently Evidenced, and Cannot Be Established. That is why the missing bridge audit report is recorded as an assurance evidence gap. It is not proof that no audit happened.
- **Five frameworks mapped as benchmarks, SOC 2 by analogy.** The crosswalk exists so a reader can see the same gap expressed in the vocabulary of NIST CSF 2.0, SP 800-53, ISO/IEC 27001:2022, COBIT 2019, and CIS Controls v8. It is a control design comparison, not a certification finding against any of them.
- **Analysis published as data, not only as prose.** The register, gap matrix, crosswalk, and roadmap are CSVs. A reader who disagrees with a score can re-sort the register and see exactly which judgements they would change.

## Worth and lessons

Most public writing about this incident stops at the exploit. The worth of this project is the part that comes after: a quantified risk register, a control environment assessment that distinguishes design gaps from evidence gaps, a remediation programme with owners and completion criteria, and an audit plan that could test whether the fix worked. That is the work a GRC function is asked to produce after an incident, and it is the work on display here.

The main lesson was discipline about measures. The incident looks simple from a distance (one bug, one attacker, one number) and gets less simple the closer the evidence is read: detection that was public rather than demonstrably internal, a bridge closed for days, a fix that arrived in two stages months apart. Writing the assessment meant refusing the tidy version whenever the sources supported only the messier one. The limitations below are part of the result, not a caveat stapled to it.

## Limitations

This is a public-evidence assessment. No internal BNB Chain or Binance documents were available, and there were no personnel interviews or system access. Where that limits a conclusion, the report says so. Nothing here asserts that any law or regulation was violated. Framework mappings are benchmarks for control design, not certification findings.

## Disclaimer

An independent portfolio assessment prepared for professional demonstration. Not affiliated with, commissioned by, or endorsed by Binance or BNB Chain. All incident facts are drawn from the cited public sources.

## License

MIT: see `LICENSE`.
