# Project BLACKSWAN: Internal Audit and Control Testing Plan
## Cross-Chain Bridge and Consensus-Critical Controls

*Prepared by Internal Audit, working independently of the teams that own these controls. Status: programme design, put forward for governance approval. The plan follows the October 2022 BSC Token Hub exploit, where forged IAVL proofs led to the release of 2,000,000 BNB (Coinbase Threat Intelligence, 31 Oct 2022, https://www.coinbase.com/en-gb/blog/bsc-token-hub-compromise-investigation-and-analysis-8).*

---

## 1. Introduction

### 1.1 Objectives
1. Give the governance body an independent view, based on evidence we have actually seen, of whether the controls around cross-chain value transfer and consensus-critical code are well designed and whether they work in practice.
2. Test this ourselves rather than take management's word for it. The point is to check that the October 2022 failure modes cannot come back unnoticed: forged proofs being accepted, issuance with no upper bound, privileged admission with no review, consensus-critical changes going live without a gate, and anomalous issuance that nobody monitors.
3. Check the remediation actions agreed in the Project BLACKSWAN roadmap (`data/remediation-roadmap.csv`). Each one has to be in place and working, judged against the Evidence of Completion set for that recommendation.
4. Report what we find against criteria agreed in advance. Critical exceptions go to the governance body within five business days, not in the next quarterly cycle.

### 1.2 Scope
This plan covers the cross-chain module (CrossChain, TokenHub, RelayerHub system contracts), the proof-verification precompile and the libraries it inherits, privileged access for relayers and validators, change management for consensus-critical code, monitoring of bridge security, and readiness to respond if the bridge is compromised. Out of scope: Binance the centralised exchange (a separate entity, and it did not operate the bridge), end-user wallet software, and the internal controls of third-party DeFi integrators. Those integrators are covered only at the point where they meet the bridge, through circuit-breaker and freeze-liaison agreements.

### 1.3 Standards basis
We built the programme around the IIA's Global Internal Audit Standards (independence, risk-based planning, sufficiency of evidence), the assessment procedures in NIST SP 800-53A (examine, interview and test), and the control objectives mapped in Section 8 of the main report (NIST CSF 2.0, ISO/IEC 27001:2022, COBIT 2019, CIS Controls v8). One point matters in practice. Where a control is automated and deterministic, as proof verification is, we reperform it. The auditor runs the control again on selected inputs. Screenshots of it running are not evidence.

### 1.4 Sampling philosophy (applies to all tests)
- **Start with the money:** we examine 100% of cross-chain releases above USD 1M equivalent in the period. Below that threshold we draw statistical attribute samples.
- **Zero tolerance on some attributes:** for attributes where a single failure is the incident itself (a forged proof accepted, a consensus-critical deployment with no gate), one occurrence is enough to raise an exception. Sampling theory does not soften that.
- **Small populations get looked at in full.** We do not sample down governance artefacts such as drills, reviews and admissions.
- **The whole period counts:** conclusions on operating effectiveness cover the full audit period, not a single day. Evidence has to exist across the period, not just in the run-up to the audit date.

---

## 2. Test Procedures

### TEST 1: Smart Contract Verification (bridge system contracts)

- **Audit Objective:** Check that the bridge system contracts (CrossChain, TokenHub, RelayerHub) running in production are the versions that were independently audited, and that no unaudited change is live.
- **Risk Statement:** If a system contract is unaudited, or gets changed quietly, it can bring back issuance flaws on the scale of R-01/R-10. Governance reporting that tracks version numbers instead of deployed bytecode would not spot it.
- **Control Objective:** Only versions that have passed the required review and audit gate go live, and the deployed bytecode can be traced back, reproducibly, to the source that was reviewed.
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

### TEST 2: Cryptographic Proof Validation

- **Audit Objective:** Reperform the proof-verification control ourselves and check that it rejects forged proofs across the audit period, including the exact pattern used in October 2022.
- **Risk Statement:** In 2022 the flaw let the hash of an injected payload bypass root-hash computation (Immunefi, 17 Mar 2023, https://immunefi.com/blog/bug-fix-reviews/hack-analysis-binance-bridge-october-2022/). A regression, or a similar flaw in the replacement verifier (ICS23), would bring back the risk of unlimited issuance (R-01, R-02).
- **Control Objective:** Every cross-chain package that is accepted has a proof that verifies under the current specification. No class of input is exempt from full verification.
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
- **Exception Criteria:** A single forged or non-verifying proof accepted, whether in testing or in the production sample, is a **Critical** exception. Any regression case absent from the suite that corresponds to a publicly disclosed proof-forgery technique is a High exception.
- **Reporting:** Immediate escalation for Critical; otherwise quarterly assurance report with reperformance statistics.

### TEST 3: Privileged Access: Relayers and Validators

- **Audit Objective:** Confirm that privileged admission to bridge functions (relayer registration, validator status) is restricted to approved parties and reviewed on cycle.
- **Risk Statement:** Before the incident anyone could register as a relayer simply by depositing 100 BNB. The attacker did exactly that at 16:55 UTC on 6 Oct 2022, ~90 minutes before the first forged withdrawal (Coinbase Threat Intelligence, 31 Oct 2022). If privileged admission is open, or admissions are never reviewed, there is no barrier left before execution to stop a malicious package being submitted (R-05).
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

### TEST 4: Validator Authorization and Emergency Authority

- **Audit Objective:** Check that validator-set changes follow the governed election and onboarding process, and that emergency pause authority is documented, has validator consent behind it, and gets practised in drills.
- **Risk Statement:** In October 2022 the chain was paused by contacting 26 active validators individually, with no documented authority in place beforehand. It worked once. It is not a repeatable control (BNB Chain, "BNB Chain Ecosystem Update," 7 Oct 2022; R-06, R-07).
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

### TEST 5: Change Management for Consensus-Critical Code

- **Audit Objective:** Confirm that changes to consensus-critical components (verifier, system contracts, consensus code, and their inherited libraries) pass the mandated review gate before deployment.
- **Risk Statement:** The code that was exploited came from an inherited library. Unless dependency and verifier changes are treated as consensus-critical and put through a gate, a routine change could introduce the same kind of flaw, or bring it back (R-10, R-02).
- **Control Objective:** 100% of consensus-critical changes receive dual review, security sign-off and, for verifier and consensus changes, an audit trigger assessment, with emergency changes reviewed after the fact.
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

### TEST 6: Security Monitoring and Anomaly Detection

- **Audit Objective:** Verify that monitoring over bridge invariants (issuance vs locked value), proof anomalies and privileged behaviour operates continuously and alerts internally within target times.
- **Risk Statement:** Detection in October 2022 came from the public and the community. The operator published no detection timeline, and a second forged withdrawal still went through over two hours after the first (Coinbase Threat Intelligence, 31 Oct 2022; samczsun, 6 Oct 2022, https://threadreaderapp.com/scrolly/1578167198203289600) (R-04).
- **Control Objective:** Anomalous issuance or proof behaviour raises an internal alert within 15 minutes, 24×7, independent of public channels, and every alert is dispositioned on the record.
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

### TEST 7: Incident Response Readiness (Bridge Compromise)

- **Audit Objective:** Assess whether the bridge-compromise playbook is current, properly resourced and can actually be executed. That includes the automatic and any-validator pause paths, issuer freeze liaison, and communications.
- **Risk Statement:** The 2022 response worked because identifiable individuals improvised. That is not a control. Readiness that depends on specific people, on authority that was never written down, or on third-party goodwill that has never been tested will not hold up (R-07, R-11, R-12).
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

The two disciplines complement each other. They are **not interchangeable**. This programme uses both, and it never treats a conclusion from one as if it came from the other.

| Dimension | Technical penetration testing | GRC control testing (this plan) |
|---|---|---|
| Core question | "Can a skilled adversary break this, today?" | "Is the control designed to address the defined risk, and is there evidence it operated throughout the period?" |
| Output | Exploitable findings at a point in time | Assurance opinion over design and operating effectiveness across a period |
| Method | Adversarial, creative, scope-flexible exploitation | Procedures, populations, sampling and exception criteria agreed in advance; reperformance where the control is deterministic |
| Evidence standard | Proof of exploit | Sufficient, appropriate evidence that the control operated, including on the 364 days nobody attacked |
| Failure meaning | A breakable path exists now | The organisation cannot show it would reliably prevent or detect the defined risk |
| Ownership | Security engineering / external testers | Internal Audit, independent of the control owners |

Take this class of incident. Penetration testing aimed at the application contracts could run for years and never touch the hash computation in an inherited library, which is where the 2022 flaw sat. Control testing has the opposite blind spot: if it never reperforms proof validation, it could certify a broken verifier simply because the paperwork looked right. So Tests 2 and 6 put technical validation (independent re-verification, injected anomalies, forged-proof regression) *inside* the audit programme. The roadmap (REC-19) also commissions adversarial testing and formal verification as management activities, and Internal Audit then evaluates the results. The assurance conclusion, on design adequacy and operating effectiveness over time, stays with Internal Audit. The exploit conclusion stays with the testers. We do not report one as a substitute for the other to the governance body.
