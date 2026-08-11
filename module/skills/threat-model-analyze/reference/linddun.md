# LINDDUN Privacy Threat Modeling Framework

LINDDUN is a systematic privacy threat modeling framework developed at KU Leuven. It provides seven privacy-specific threat categories, complementing security-focused frameworks like STRIDE. Use LINDDUN when the system processes personal data and privacy is a primary concern — especially under GDPR, CCPA, HIPAA, or similar regulations.

## Threat Categories

### Linkability
The ability to determine that two or more items of interest (data records, actions, users) are related — without necessarily knowing the identity.

**Questions:**
- Can an attacker link records across datasets to build profiles?
- Can browsing patterns, transaction histories, or usage logs be correlated?
- Can pseudonymized data be re-linked through quasi-identifiers?
- Are there shared identifiers (email, device ID, IP) across services that enable cross-system linking?

**Examples:** Linking shopping history with health data via shared loyalty card ID; correlating anonymized GPS traces to identify home/work locations; connecting pseudonymized records across microservices.

**Mitigations:** Data minimization, purpose limitation, pseudonymization with different keys per context, unlinkable credentials, mix networks, differential privacy.

### Identifiability
The ability to identify a specific individual from data — moving from "someone" to "this person."

**Questions:**
- Can individuals be identified from supposedly anonymous datasets?
- Are there enough quasi-identifiers (age, zip code, gender) to re-identify?
- Can metadata (timestamps, access patterns, IP addresses) reveal identity?
- Does the system collect or expose unique identifiers unnecessarily?

**Examples:** Re-identifying Netflix users from anonymized viewing history; identifying individuals from "anonymous" location data; fingerprinting browsers or devices.

**Mitigations:** k-anonymity, l-diversity, t-closeness, differential privacy, data aggregation, removing quasi-identifiers, avoiding unique device/browser fingerprints.

### Non-repudiation (Privacy Context)
In privacy, non-repudiation is a *threat* — the inability to deny having performed an action. Unlike security (where non-repudiation is desired), privacy requires plausible deniability for certain actions.

**Questions:**
- Can users deny having performed sensitive actions (visiting a health site, filing a complaint)?
- Do audit logs expose more about user behavior than necessary for their purpose?
- Can digital signatures or blockchain records make it impossible for a user to deny participation?
- Are there situations where users should be able to act anonymously but cannot?

**Examples:** Whistleblower systems that log reporter identity; health portal access logs that reveal which conditions a user researched; voting systems that link votes to voters.

**Mitigations:** Anonymous credentials, zero-knowledge proofs, purpose-limited logging (log events without actor identity where possible), privacy-preserving audit trails.

### Detectability
The ability to determine whether an item of interest (a record, a user, a communication) exists — even without accessing its content.

**Questions:**
- Can an attacker determine that a specific user has an account?
- Can traffic analysis reveal that communication is happening (even if encrypted)?
- Do API responses differ between "not found" and "not authorized" (leaking existence)?
- Can database query timing or response size reveal whether matching records exist?

**Examples:** Username enumeration via login error messages; traffic analysis revealing VPN usage; timing side-channels in encrypted database queries; HTTP response size differences revealing content existence.

**Mitigations:** Uniform error responses, constant-time operations, traffic padding, dummy records, oblivious RAM (ORAM), private information retrieval.

### Disclosure of Information (Privacy Context)
The exposure of personal data to unauthorized parties — similar to STRIDE's Information Disclosure but focused specifically on personal/sensitive data and privacy implications.

**Questions:**
- Is personal data exposed beyond what's necessary for the stated purpose?
- Can data processors or subprocessors access more personal data than needed?
- Are there data retention policies, and is data actually deleted when it should be?
- Can personal data leak through logs, error messages, analytics, or backups?
- Is data shared with third parties beyond what users consented to?

**Examples:** Analytics platforms receiving user PII; backup systems retaining data past retention periods; verbose error messages exposing user details; over-scoped API tokens accessing personal data.

**Mitigations:** Data minimization, purpose limitation, encryption, access controls with purpose binding, automated retention/deletion, consent management, data processing agreements.

### Unawareness
Users not being adequately informed about or in control of the collection, processing, and sharing of their personal data.

**Questions:**
- Do users know what data is collected about them?
- Can users access, correct, or delete their personal data (data subject rights)?
- Are privacy policies clear, accessible, and honest about data practices?
- Are users notified when their data is breached?
- Can users understand and control consent decisions?
- Are there dark patterns that manipulate users into sharing more data?

**Examples:** Hidden data collection via trackers; privacy policies written in impenetrable legalese; no mechanism for data export or deletion; automatic opt-in to data sharing; no breach notification process.

**Mitigations:** Transparency reports, privacy dashboards, granular consent controls, plain-language privacy notices, data subject access request (DSAR) workflows, breach notification procedures, privacy by design.

### Non-compliance
Failure to comply with privacy legislation, regulations, standards, or organizational policies.

**Questions:**
- Does the system comply with applicable privacy regulations (GDPR, CCPA, HIPAA, PIPEDA)?
- Are data processing activities documented and lawful (legal basis for processing)?
- Is there a Data Protection Impact Assessment (DPIA) where required?
- Are cross-border data transfers handled correctly (adequacy decisions, SCCs, BCRs)?
- Are data processors and subprocessors contractually bound?
- Is there a designated DPO where required?

**Examples:** Processing personal data without valid legal basis; transferring EU data to non-adequate countries without safeguards; failing to conduct required DPIAs for high-risk processing; missing records of processing activities.

**Mitigations:** Privacy compliance program, records of processing activities (ROPA), DPIA process, data processing agreements, transfer impact assessments, DPO appointment, regular compliance audits.

## How to Apply LINDDUN

1. **Model the system**: Create a data flow diagram focusing on personal data flows.
2. **Map data subjects**: Identify whose personal data flows through the system and in what form.
3. **Per-element analysis**: For each DFD element (process, data store, data flow, external entity), consider each LINDDUN category.
4. **Elicit threats**: Use the LINDDUN threat tree catalogs to identify specific privacy threats.
5. **Prioritize**: Rate threats by privacy impact (severity of harm to data subjects) and likelihood.
6. **Select mitigations**: Map threats to privacy-enhancing technologies (PETs) and organizational measures.

## When to Use LINDDUN

- The system processes personal data (PII, PHI, financial data)
- GDPR, CCPA, HIPAA, or similar regulations apply
- Privacy is a competitive differentiator or brand promise
- The system involves profiling, tracking, or behavioral analysis
- Data is shared across organizational boundaries

## LINDDUN + STRIDE

LINDDUN and STRIDE are complementary. STRIDE covers security threats; LINDDUN covers privacy threats. For systems that handle personal data, run both — many threats (like Information Disclosure) appear in both but with different lenses (security: "can an attacker access it?" vs. privacy: "should this data be collected at all?").
