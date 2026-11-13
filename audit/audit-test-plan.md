# Project BLACKSWAN — Internal Audit and Control Testing Plan
## Cross-Chain Bridge and Consensus-Critical Controls

*Prepared by: Internal Audit (independent assurance). Status: programme design for governance approval. This plan responds to the October 2022 BSC Token Hub exploit, in which forged IAVL proofs caused the release of 2,000,000 BNB (Coinbase Threat Intelligence, 31 Oct 2022, https://www.coinbase.com/en-gb/blog/bsc-token-hub-compromise-investigation-and-analysis-8).*

---

## 1. Introduction

### 1.1 Objectives
1. Provide independent, evidence-based assurance over the design adequacy and operating effectiveness of controls protecting cross-chain value transfer and consensus-critical code.
2. Verify — by testing, not by accepting management assertion — that the specific failure modes of October 2022 cannot recur undetected: forged proof acceptance, unbounded issuance, unreviewed privileged admission, ungated consensus-critical change, and unmonitored anomalous issuance.
3. Confirm that remediation actions accepted under the Project BLACKSWAN roadmap (`data/remediation-roadmap.csv`) are implemented and operating, using the Evidence of Completion defined for each recommendation.
4. Report exceptions against pre-agreed criteria to the governance body, with critical exceptions escalated within five business days.

### 1.2 Scope
In scope: the cross-chain module (CrossChain, TokenHub, RelayerHub system contracts), the proof-verification precompile and its inherited libraries, relayer and validator privileged access, consensus-critical change management, bridge security monitoring, and incident-response readiness for bridge compromise. Out of scope: Binance the centralised exchange (a separate entity that did not operate the bridge), end-user wallet software, and third-party DeFi integrators' internal controls (addressed only at the interface — circuit-breaker and freeze-liaison agreements).

### 1.3 Standards basis
The programme is designed by reference to the IIA's Global Internal Audit Standards (independence, risk-based planning, evidence sufficiency), NIST SP 800-53A assessment procedures (examine / interview / test methods), and the control objectives mapped in Section 8 of the main report (NIST CSF 2.0, ISO/IEC 27001:2022, COBIT 2019, CIS Controls v8). Where a control is automated and deterministic (proof verification), testing follows the reperformance model: the auditor independently executes the control on selected inputs rather than inspecting screenshots of its operation.

### 1.4 Sampling philosophy (applies to all tests)
- **Value-weighted completeness:** 100% of cross-chain releases above USD 1M equivalent in the period are examined; statistical attribute samples are drawn below that threshold.
- **Zero-tolerance attributes:** for attributes where one failure equals the incident (a forged proof accepted; an ungated consensus-critical deployment), the exception criterion is a single occurrence — sampling theory does not dilute this.
- **Small populations are examined in full.** Governance artefacts (drills, reviews, admissions) are never sampled down.
- **Period coverage:** operating-effectiveness conclusions cover the full audit period, not a point in time; evidence must exist across the period, not only near the audit date.

---

## 2. Test Procedures

### TEST 1 — Smart Contract Verification (bridge system contracts)

- **Audit Objective:** Confirm that the bridge system contracts (CrossChain, TokenHub, RelayerHub) in production are the independently audited code versions, and that no unaudited modification is live.
- **Risk Statement:** Unaudited or silently modified system contracts can reintroduce issuance flaws equivalent to R-01/R-10, undetectable by governance reporting that tracks versions rather than deployed bytecode.
- **Control Objective:** Only code versions that have passed the mandated review and audit gate are deployed; deployed bytecode is reproducibly traceable to reviewed source.
- **Audit Procedure:**
  1. Obtain the register of consensus-critical components and their approved source versions for the period.
  2. Independently fetch deployed bytecode for each in-scope contract from a full node / explorer and compare hashes against the approved build artefacts.
  3. For each version change in the period, trace to its change record, review evidence and audit status (link to Test 5).
  4. Confirm the most recent independent audit report covers the currently deployed versions; list any deployed version with no corresponding audit coverage.
  5. Inspect the findings tracker: confirm no Critical audit finding is open past its remediation date without a risk-committee-approved exception.
- **Population:** All bridge system contracts and all version deployments in the audit period.
- **Sampling Approach:** 100% of contracts; 100% of deployments (population is small).
- **Evidence Required:** Contract register; build artefacts and hash comparison output; deployment records; audit reports; findings tracker extract.
- **Expected Result:** Every deployed version hashes to an approved artefact and is covered by an audit report or an approved, dated exception.
- **Exception Criteria:** Any deployed version not traceable to reviewed source (Critical); any Critical audit finding overdue without approved exception (High).
- **Reporting:** Results to the governance body in the quarterly assurance report; Critical exceptions within five business days.

### TEST 2 — Cryptographic Proof Validation

- **Audit Objective:** Verify that the proof-verification control rejects forged proofs — including the exact October 2022 attack pattern — across the audit period, by independent reperformance.
- **Risk Statement:** The 2022 flaw let an injected payload's hash bypass root-hash computation (Immunefi, 17 Mar 2023, https://immunefi.com/blog/bug-fix-reviews/hack-analysis-binance-bridge-october-2022/); a regression or a sibling flaw in the replacement verifier (ICS23) would restore unlimited issuance risk (R-01, R-02).
- **Control Objective:** Every accepted cross-chain package carries a proof that verifies under the current specification, with no input class exempt from full verification.
- **Audit Procedure:**
  1. Obtain the verifier specification and the forged-proof regression suite; confirm the suite includes: the historical 2022 pattern (payload in the unused right attribute; anomalous historical block height; abnormally short proof), malformed/empty inner nodes, and duplicate-leaf constructions.
  2. Execute the regression suite independently against the production-equivalent verifier build; record pass/fail per case.
  3. Select the transaction sample per §1.4 (100% above USD 1M equivalent; statistical sample below) and independently re-verify each sampled package's proof against source-chain state.
  4. For any proof accepted in production that fails independent re-verification, halt testing, preserve evidence, and escalate as a critical exception immediately.
  5. Review the dependency inventory for the verifier's libraries (link to roadmap REC-09): confirm versions are pinned and that upstream critical disclosures in the period received applicability assessments within SLA.
- **Population:** All cross-chain packages accepted in the period; the full regression suite.
- **Sampling Approach:** 100% of regression cases; value-weighted transaction sampling per §1.4 (100% > USD 1M; minimum 60 items below, randomly selected).
- **Evidence Required:** Verifier specification; regression suite and execution logs; sampled package data and independent re-verification output; dependency inventory and disclosure assessment records.
- **Expected Result:** All regression cases rejected; all sampled production proofs independently verify.
- **Exception Criteria:** A single forged or non-verifying proof accepted — in test or in the production sample — is a **Critical** exception. Any regression case absent from the suite that corresponds to a publicly disclosed proof-forgery technique is a High exception.
- **Reporting:** Immediate escalation for Critical; otherwise quarterly assurance report with reperformance statistics.

### TEST 3 — Privileged Access: Relayers and Validators

- **Audit Objective:** Confirm that privileged admission to bridge functions (relayer registration, validator status) is restricted to approved parties and reviewed on cycle.
- **Risk Statement:** Before the incident, anyone could register as a relayer by depositing 100 BNB; the attacker did so at 16:55 UTC on 6 Oct 2022, ~90 minutes before the first forged withdrawal (Coinbase Threat Intelligence, 31 Oct 2022). Open or unreviewed privileged admission removes the only pre-execution barrier to submitting malicious packages (R-05).
- **Control Objective:** Relayer and validator entitlements are granted only under documented criteria, are attributable to accountable parties, and are re-certified at least quarterly.
- **Audit Procedure:**
  1. Extract the full list of active relayers and validators at period end and all admissions during the period.
  2. For each admission, trace to its approval record against the published admission criteria; flag any admission lacking evidence of vetting.
  3. Inspect the registration mechanism configuration: confirm open self-registration remains disabled, or that compensating controls approved by governance are in force with a current re-justification date for the interim allow-list.
  4. Obtain quarterly re-certification records; confirm 100% of entitlements were reviewed and that removals identified were executed on-chain within the agreed window.
  5. Identify dormant privileged entitlements (no activity in 90 days) and confirm they were challenged in the review.
- **Population:** All active relayers and validators; all admissions and removals in the period.
- **Sampling Approach:** 100% examination (population is small and each item is high-privilege).
- **Evidence Required:** On-chain entitlement extracts; admission approval records; configuration evidence; re-certification packs; removal transaction references.
- **Expected Result:** Every entitlement is approved, attributable and in-date for review.
- **Exception Criteria:** Any active entitlement without an approval record (Critical); re-certification completion below 100% (High); interim allow-list past its re-justification date (Medium).
- **Reporting:** Quarterly assurance report; Critical exceptions within five business days.

### TEST 4 — Validator Authorization and Emergency Authority

- **Audit Objective:** Verify that validator-set changes follow the governed election/onboarding process, and that emergency pause authority is documented, consented and exercised in drills.
- **Risk Statement:** In October 2022 the chain was paused by contacting 26 active validators individually, with no pre-existing documented authority — effective once, but not a repeatable control (BNB Chain, "BNB Chain Ecosystem Update," 7 Oct 2022; R-06, R-07).
- **Control Objective:** Validator authorization changes are governed and transparent; a defined quorum can execute an emergency pause under pre-agreed authority within the target time.
- **Audit Procedure:**
  1. Reconcile validator-set changes in the period to governance/election records; investigate any change without a corresponding record.
  2. Inspect the emergency authority charter: confirm named roles, quorum definition, and validator consent records are current.
  3. Review drill evidence for the period: confirm at least the scheduled exercises occurred, measure decision-to-pause time against the target, and confirm lessons were tracked to closure.
  4. Test the 24×7 contact protocol by inspecting the contact registry's last verification date and a sample of validator acknowledgements.
  5. Where any-validator channel pause is implemented, confirm drill evidence that a single validator executed a test pause within target time and that misuse safeguards (logging, notification, review) operated.
- **Population:** All validator-set changes; all drills in the period; the current charter and registry.
- **Sampling Approach:** 100% of changes and drills; sample of 10 validator contact acknowledgements (or all if fewer).
- **Evidence Required:** Governance/election records; charter; consent registry; drill reports with timestamps; contact verification logs.
- **Expected Result:** All changes governed; drills meet time targets; authority documentation current.
- **Exception Criteria:** Any ungoverned validator change (Critical); a missed drill cycle or pause decision exceeding twice the target time (High).
- **Reporting:** Quarterly assurance report; annual governance opinion input.

### TEST 5 — Change Management for Consensus-Critical Code

- **Audit Objective:** Confirm that changes to consensus-critical components (verifier, system contracts, consensus code, and their inherited libraries) pass the mandated review gate before deployment.
- **Risk Statement:** The exploited code was inherited library code; without a gate that treats dependency and verifier changes as consensus-critical, an equivalent flaw can be introduced or reintroduced by routine change (R-10, R-02).
- **Control Objective:** 100% of consensus-critical changes receive dual review, security sign-off, and — for verifier/consensus changes — an audit trigger assessment, with emergency changes retrospectively reviewed.
- **Audit Procedure:**
  1. Obtain the consensus-critical component register and the complete list of releases touching those components in the period (from repository and deployment records, independently extracted).
  2. For each release, trace to: dual-review evidence, security sign-off, audit-trigger assessment, and deployment approval.
  3. Identify emergency changes; confirm each received retrospective review within the mandated window and that the retrospective concluded on the record.
  4. Reconcile the release list to deployed versions from Test 1; investigate any deployed version with no corresponding gated release.
  5. Review the exception log: confirm every exception carries governance approval and an expiry date, and none is expired.
- **Population:** All releases touching consensus-critical components in the period.
- **Sampling Approach:** 100% of verifier/consensus releases; if volume is high, 100% of emergency changes plus a random sample of 25 standard changes.
- **Evidence Required:** Component register; repository history; review and sign-off records; deployment logs; emergency retrospectives; exception log.
- **Expected Result:** Complete gate compliance; zero unexplained deployments.
- **Exception Criteria:** Any consensus-critical deployment without the mandated gate (Critical); emergency change without timely retrospective (High); expired exception still relied upon (Medium).
- **Reporting:** Quarterly assurance report; Critical exceptions within five business days; KRI 7 data to the dashboard.

### TEST 6 — Security Monitoring and Anomaly Detection

- **Audit Objective:** Verify that monitoring over bridge invariants (issuance vs locked value), proof anomalies and privileged behaviour operates continuously and alerts internally within target times.
- **Risk Statement:** In October 2022 detection was public and community-driven; the operator published no detection timeline, and a second forged withdrawal succeeded over two hours after the first (Coinbase Threat Intelligence, 31 Oct 2022; samczsun, 6 Oct 2022, https://threadreaderapp.com/scrolly/1578167198203289600) (R-04).
- **Control Objective:** Anomalous issuance or proof behaviour generates an internal alert within 15 minutes, 24×7, independent of public channels, and every alert is dispositioned on the record.
- **Audit Procedure:**
  1. Inspect the monitoring rule set: confirm rules cover unmatched issuance, stale/anomalous block-height references, abnormal proof characteristics, burst issuance, and dormant-relayer activation.
  2. Review the alert log for the period: confirm every alert has a disposition, an owner and a time-to-acknowledge; compute MTTD statistics for the dashboard (KRI 3).
  3. With prior notice to the governance body but not to the operations team, run (or observe) an injected test anomaly in a production-like environment and measure detection and paging end to end.
  4. Confirm no incident in the period was first detected via a public/third-party channel; if any was, treat as a critical exception and require a root-cause action.
  5. Inspect on-call rosters and paging test records for the full period for gaps in coverage.
- **Population:** All production alerts in the period; all monitoring rules; on-call coverage for the period.
- **Sampling Approach:** 100% of alerts above the L1 threshold; random sample of 30 lower-severity alerts; one live-fire detection test per cycle.
- **Evidence Required:** Rule configurations; alert and disposition logs; test-anomaly results; rosters and paging tests.
- **Expected Result:** Detection within target; complete dispositioning; no public-first detections.
- **Exception Criteria:** Any real incident first detected publicly (Critical); injected test anomaly undetected within 60 minutes (High); undispositioned L1+ alerts (Medium).
- **Reporting:** Quarterly assurance report with MTTD trend; Critical exceptions immediately.

### TEST 7 — Incident Response Readiness (Bridge Compromise)

- **Audit Objective:** Assess whether the bridge-compromise playbook is current, resourced and demonstrably executable — including automatic and any-validator pause paths, issuer freeze liaison, and communications.
- **Risk Statement:** The 2022 response succeeded through improvisation by identifiable individuals; readiness that depends on specific people, undocumented authority or untested third-party goodwill is not a control (R-07, R-11, R-12).
- **Control Objective:** A documented, exercised playbook exists with defined escalation thresholds; pause, freeze-liaison and communication actions execute within target times in exercises.
- **Audit Procedure:**
  1. Inspect the playbook: confirm it contains the escalation thresholds, decision rights, issuer-liaison pack, integrator circuit-breaker contacts and pre-approved communication templates; confirm version currency and ownership.
  2. Review exercise records for the period (tabletop and functional): attendance, scenario coverage (must include forged-proof issuance and lending-market contagion), objectives met, and lesson tracking.
  3. Observe one functional exercise component: measure confirm-to-pause time (MTTC) against target and confirm the automatic circuit breaker triggers without human approval at the L2 threshold (link to roadmap REC-02).
  4. Test the freeze-liaison protocol: confirm contact drills with major issuers occurred within the period and acknowledgements met the target time.
  5. Confirm the post-incident documentation standard: the playbook requires a timestamped public chronology for L3+ events, closing the gap observed in 2022 when neither official post stated exact halt times.
- **Population:** Current playbook; all exercises and liaison drills in the period.
- **Sampling Approach:** 100% of exercises and drills (small population); observation of one functional exercise per annual cycle.
- **Evidence Required:** Playbook and version history; exercise reports with measured times; liaison drill logs; communication templates.
- **Expected Result:** Playbook current; exercise objectives ≥ 90% met; MTTC within target; liaison drills in date.
- **Exception Criteria:** No functional exercise in two consecutive quarters (High); automatic pause failing in exercise (Critical); freeze-liaison contacts untested for over six months (Medium).
- **Reporting:** Semi-annual readiness opinion to the governance body; Critical exceptions immediately.

---

## 3. Penetration Testing vs GRC Control Testing

These two disciplines are complementary and **not interchangeable**; this programme uses both and never substitutes one conclusion for the other.

| Dimension | Technical penetration testing | GRC control testing (this plan) |
|---|---|---|
| Core question | "Can a skilled adversary break this, today?" | "Is the control designed to address the defined risk, and is there evidence it operated throughout the period?" |
| Output | Exploitable findings at a point in time | Assurance opinion over design and operating effectiveness across a period |
| Method | Adversarial, creative, scope-flexible exploitation | Pre-agreed procedures, populations, sampling and exception criteria; reperformance where the control is deterministic |
| Evidence standard | Proof of exploit | Sufficient, appropriate evidence that the control operated — including for the 364 days nobody attacked |
| Failure meaning | A breakable path exists now | The organisation cannot demonstrate it would prevent/detect the defined risk reliably |
| Ownership | Security engineering / external testers | Internal Audit, independent of the control owners |

Applied to this incident class: penetration testing scoped to application contracts could run for years without exercising an inherited library's hash computation (the 2022 flaw), while control testing that never reperforms proof validation would certify a broken verifier on the strength of its documentation. Accordingly, Tests 2 and 6 embed technical validation (independent re-verification, injected anomalies, forged-proof regression) *inside* the audit programme, and the roadmap (REC-19) commissions adversarial testing and formal verification as management activities whose results Internal Audit then evaluates. The assurance conclusion — design adequacy and operating effectiveness over time — remains with Internal Audit; the exploit conclusion remains with the testers. Neither is accepted as a substitute for the other in reporting to the governance body.
