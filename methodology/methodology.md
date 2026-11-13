# Project BLACKSWAN — Assessment Methodology

*How the Enterprise GRC Assessment of the 2022 BNB Chain Bridge Exploit was performed. Companion to the main report; the evidence index is at `evidence/evidence-and-sources.md`.*

## 1. Scope

**In scope:** the BSC Token Hub cross-chain bridge between BNB Beacon Chain and BNB Smart Chain; the incident of 6–12 October 2022 and its documented aftermath through the BEP-171 implementation (Planck hard fork, 12 April 2023); the governance, risk, control, assurance and incident-response arrangements of BNB Chain (protocol, core development community, validator set) as evidenced publicly.

**Out of scope:** Binance the centralised exchange's internal controls (a separate entity; it did not operate the bridge); criminal investigation or attribution; smart-contract code audit (this is a GRC assessment that *evaluates* assurance arrangements, not a re-audit); any non-public information.

**Entity discipline:** every finding is assigned to BNB Chain / the bridge unless evidence specifically supports otherwise. Conflation of BNB Chain with Binance CEX is treated as an assessment error and is itself tracked as a risk (R-16).

## 2. Evidence sources and hierarchy

Evidence was gathered from public sources only, weighted by the following hierarchy (highest first):

1. **Primary operator artefacts** — BNB Chain's own publications (7 Oct 2022 Ecosystem Update; 11 Oct 2022 "A Decentralized Response") and public code artefacts (bnb-chain/bsc release v1.1.16). Treated as authoritative for the operator's stated positions and admissions; flagged as self-interested where the operator characterises its own decentralisation or performance.
2. **Independent technical analyses with on-chain evidence** — Coinbase Threat Intelligence (31 Oct 2022), Immunefi (17 Mar 2023), Elliptic, SlowMist, and the contemporaneous analysis of samczsun (Paradigm, 6 Oct 2022). Treated as authoritative for transaction-level facts where two or more agree.
3. **Reputable contemporaneous reporting** — CoinDesk, The Block, Decrypt, Cointelegraph, CNBC, Blockworks, Messari. Used for timestamps, statements and market context; any figure carried by only one outlet is attributed to that outlet.
4. **Secondary / low-authority sources** — used only to document that a claim circulates (e.g., Callisto Network's assertion that no bridge audit existed; FXStreet's claim that PeckShield was the first detector), never as the sole basis for a finding. Such claims are labelled unverified wherever they appear.

The full catalogue, with publication dates and URLs, is the evidence index (`evidence/evidence-and-sources.md`, entries E-01 onward).

## 3. Citation standard

Every material factual assertion carries an inline citation in the form *(Source, date, URL)*. On-chain timestamps are stated in UTC. Where the underlying page could not be retrieved directly (the BNB Chain Ecosystem Update of 7 Oct 2022), the citation names the corroborating outlets through which its content was verified.

## 4. Fact vs interpretation labelling

- **Verified fact:** supported by at least one Tier 1–2 source, or two independent Tier 3 sources. Stated without qualification.
- **Conflicting figure:** where sources disagree, the report presents the attributed range and never silently selects a value (see the Conflicting Figures Register in the evidence index). The three-number convention is mandatory throughout: gross forged value (~$566M–$586M), value moved off-chain (~$70M–$127M), operator's final unrecovered figure (~$100M).
- **Interpretation:** analytical judgements are labelled **[Interpretation]** at first use in each section, or are grammatically framed as assessment ("this assessment records…"). Interpretations are never presented in the evidence index as facts.
- **Unverified claim:** single-source or low-authority claims are labelled as such at the point of use.

## 5. Risk scoring method

Risks (register, Section 6; IDs R-01 to R-16) are scored on a **5×5 matrix**: Likelihood (1 Rare – 5 Almost Certain) × Impact (1 Minimal – 5 Catastrophic), assessed *inherently* (before the recommended remediation, reflecting the position evidenced at/around the incident) and *residually* (assuming the mapped roadmap recommendations are implemented and operating).

**Bands:** 1–4 **Low** · 5–9 **Moderate** · 10–16 **High** · 17–25 **Extreme**.

Impact criteria combine financial exposure (anchored to the incident's realised scale: a scenario capable of nine-figure loss scores ≥ 4), operational impact (chain/bridge downtime), ecosystem contagion, and reputational/regulatory consequence. Likelihood criteria consider threat-actor capability demonstrated in the incident, exposure of the attack surface, and the pre-incident industry base rate (bridges were the dominant loss category in 2022 — Chainalysis, 2 Aug 2022). Scores are judgement-based and documented per risk in the register; they are not statistical estimates, and monetary exposure is always expressed as the attributed ranges of §4 rather than as invented point estimates.

## 6. Gap classification taxonomy

Every control gap in Section 7 is classified into exactly one primary category, because the remediation differs by category:

| Class | Meaning | Evidentiary standard |
|---|---|---|
| **Absent** | No evidence the control existed at the time of the incident | Exhaustive search of the public record + a mechanism that demonstrably did not trigger (e.g., no automatic pause occurred) |
| **Ineffective** | The control existed but did not achieve its objective under incident conditions | Incident evidence showing the control operating and failing (e.g., root-hash verification performed but defeated) |
| **Insufficiently evidenced** | The control may exist, but no public evidence demonstrates its design or operation | Absence of artefacts (reports, records, timelines) that the control would normally produce |
| **Cannot be established** | Available sources neither confirm nor refute the control's existence | Explicit statement of what evidence would resolve the question |

Classify conservatively: "Absent" requires positive reason to believe the control did not exist, not merely a missing document; documentation gaps default to "Insufficiently evidenced."

## 7. Limitations

1. **Public sources only.** No internal policies, tickets, audit working papers, post-incident reviews beyond the public posts, or personnel interviews were available. Governance findings therefore assess *demonstrable* governance; an undocumented control may have existed and cannot be credited.
2. **No detailed operator post-mortem.** The operator's 7 October 2022 post promised a fuller timeline; no standalone detailed post-mortem was located. Exact halt block height and halt start minute could not be verified; Coinbase's 22:19 UTC announcement timestamp and the ~06:40 UTC resumption are the best-supported bookends.
3. **Unresolved items** (full list in the evidence index): a single first detector; whether any private audit of the bridge occurred; the outcomes of the 7 October 2022 governance votes; a final audited funds-flow accounting for the ~$429M stranded on BSC and the attacker's lending position.
4. **Attribution:** no public attribution of the attacker exists (Coinbase: actor unknown). No finding in this assessment depends on attacker identity.
5. **Retrospective frameworks:** NIST CSF 2.0 (2024) post-dates the incident; it is used as an assessment benchmark, not as a standard the operator claimed to meet in 2022. ISO/IEC 27001:2022 was published in the same month as the incident; mappings are likewise benchmark mappings, and no certification failure is asserted anywhere in this assessment.
6. **Self-interested sources:** operator statements about decentralisation and performance are used as admissions/positions, corroborated where possible against independent analyses; a critic's allegation that the validator set was effectively controlled by the affiliated exchange (reported 17 Oct 2022) is recorded as an unverified allegation, not a finding.

## 8. Quality assurance review approach

Before issue, the draft underwent a structured challenge review against the following questions, with corrections made where the draft failed them:

1. Is every factual claim traceable to a catalogued source, and is every number inside its attributed range?
2. Does any passage present a technical weakness as a *confirmed governance failure* without governance evidence? (Downgraded to interpretation or limitation where found.)
3. Are BNB Chain and Binance CEX kept distinct in every section, including in quotations selected?
4. Do risk scores follow the §5 method, and does every High/Extreme risk trace to at least one roadmap recommendation and one audit test or KRI?
5. Are framework mappings (Section 8) tied to the actual attack mechanics rather than generic control language?
6. Are recommendations specific enough to be independently tested — each with an owner, success criterion and evidence of completion (see `data/remediation-roadmap.csv`)?
7. Are "could not verify" items disclosed rather than smoothed over, and is any figure traceable to a single low-authority source labelled unverified?
8. Does the conclusion (§15.6) avoid any numeric maturity score, given the public-only evidence base?

Residual QA risk is owned by the Lead GRC Engineer; the known residual weaknesses are the limitations in §7, which no amount of drafting can cure — only access to internal evidence could.
