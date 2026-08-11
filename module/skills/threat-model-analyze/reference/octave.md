# OCTAVE (Operationally Critical Threat, Asset, and Vulnerability Evaluation)

OCTAVE is a risk-based strategic assessment framework developed at Carnegie Mellon's SEI. Unlike STRIDE (technical, per-element) or PASTA (process-driven, per-application), OCTAVE evaluates security risk at the organizational level. It is self-directed — designed to be conducted by the people who own and operate the systems, not external auditors.

## OCTAVE Variants

### OCTAVE Allegro (Recommended)
The streamlined variant. Focuses on information assets and is the most practical for most teams. This is the variant this skill implements.

### OCTAVE-S
For small organizations (<100 people). Simplified worksheets.

### OCTAVE (Original)
For large organizations. Requires dedicated workshop teams across management, operations, and IT. Heavyweight.

## OCTAVE Allegro: Eight Steps

### Step 1: Establish Risk Measurement Criteria
Define how your organization measures risk before analyzing anything. This prevents subjective scoring later.

**Define impact areas and their relative priority:**

| Impact Area | Description | Priority |
|---|---|---|
| Financial | Revenue loss, fines, remediation costs | (rank 1-5) |
| Reputation | Customer trust, brand damage, media coverage | (rank 1-5) |
| Productivity | Operational disruption, employee downtime | (rank 1-5) |
| Safety | Physical safety of people | (rank 1-5) |
| Legal/Regulatory | Compliance violations, legal liability | (rank 1-5) |

For each impact area, define a scoring scale (Low=1, Medium=2, High=3) with concrete descriptions specific to the organization. Example:
- Financial High: >$1M loss
- Financial Medium: $100K–$1M loss
- Financial Low: <$100K loss

### Step 2: Develop Information Asset Profiles
Identify and profile the critical information assets (not systems — *information*).

**For each information asset, document:**
- **Asset name and description**: What is this information?
- **Owner**: Who is accountable for this information?
- **Security requirements**:
  - Confidentiality: who should have access? What's the harm of unauthorized access?
  - Integrity: what's the harm if this information is modified without authorization?
  - Availability: what's the harm if this information is unavailable? For how long?
- **Most important security requirement**: which of the three matters most for this asset?

**Examples of information assets:**
- Customer PII database
- Source code repository
- Financial reporting data
- Employee records
- API credentials and secrets
- Audit logs
- Encryption keys

### Step 3: Identify Information Asset Containers
Map where each information asset lives — its "containers." An information asset typically spans multiple containers.

**Container types:**
- **Technical**: databases, file systems, servers, cloud storage, backups, applications
- **Physical**: paper documents, portable media, data center locations
- **People**: employees, contractors, partners who know or have access to the information

For each container, note:
- Who has access (internal, external)?
- What controls exist?
- Is the asset in transit, at rest, or in use?

### Step 4: Identify Areas of Concern
For each information asset, brainstorm what could go wrong. These are scenario-based, not category-based.

**Think about real scenarios:**
- "A disgruntled employee with database access downloads the full customer table"
- "A developer accidentally commits API keys to a public repository"
- "Our cloud provider has an outage and we lose access to the application for 8 hours"
- "A phishing email compromises an admin account"

Don't filter at this stage — capture everything. Each area of concern becomes a threat scenario in the next step.

### Step 5: Identify Threat Scenarios
Formalize each area of concern into a structured threat scenario.

**Threat scenario components:**
- **Actor**: Who is the threat source? (see threat-actors.md for persona profiles)
  - Internal (employee, contractor) or External (hacker, competitor, nation-state)
  - Human or Non-human (natural disaster, system failure, software bug)
- **Motive**: Why would they do this? (deliberate, accidental, environmental)
- **Access**: How do they reach the asset? (network, physical, insider, supply chain)
- **Asset**: Which information asset is affected?
- **Outcome**: What happens? (disclosure, modification, destruction, interruption)
- **Security requirement violated**: Confidentiality, Integrity, or Availability?

### Step 6: Identify Risks
For each threat scenario, assess the risk using the criteria from Step 1.

**Score each impact area:**
- Apply the scoring scale (Low/Medium/High → 1/2/3) for each impact area
- Multiply by the priority weight from Step 1
- Sum the weighted scores for a total risk score

**Relative risk score = sum of (impact area score x impact area priority weight)**

This produces a numeric risk score that is consistent across all threat scenarios and grounded in the organization's own priorities.

### Step 7: Analyze Risks
Rank all threat scenarios by risk score and categorize:

| Risk Level | Action |
|---|---|
| High (top quartile) | Mitigate immediately — these are unacceptable risks |
| Medium | Mitigate in the near term or accept with documented rationale |
| Low | Accept, monitor, or defer |

For each high-risk scenario, also assess:
- **Probability**: How likely is this scenario? (historical data, threat intelligence)
- **Existing controls**: What's already in place to prevent or detect this?
- **Control gaps**: Where are the controls insufficient?

### Step 8: Select Mitigation Approach
For each risk requiring mitigation, choose a strategy:

| Strategy | Description | When to use |
|---|---|---|
| **Mitigate** | Implement controls to reduce likelihood or impact | Default for high risks |
| **Transfer** | Shift risk to a third party (insurance, outsourcing) | When mitigation cost exceeds risk cost |
| **Accept** | Acknowledge and document the risk | When risk is within tolerance |
| **Avoid** | Eliminate the risk by removing the asset or activity | When the activity isn't worth the risk |

For each mitigation, document:
- What control will be implemented
- Who is responsible
- Timeline for implementation
- Expected risk reduction
- Residual risk after mitigation

## When to Use OCTAVE

- Organizational-level risk assessment (not just one application)
- When business stakeholders need to understand and own security risk
- When you need to prioritize security investments across multiple systems
- Compliance-driven assessments that require documented risk acceptance
- When the organization is assessing its overall security posture
- When non-technical threats (physical, human, process) are in scope

## OCTAVE + Other Frameworks

OCTAVE provides the organizational risk context; other frameworks provide the technical depth:
- **OCTAVE → STRIDE**: OCTAVE identifies critical assets; STRIDE analyzes the technical threats to those assets
- **OCTAVE → PASTA**: OCTAVE's business context feeds directly into PASTA's Stage 1 (business objectives)
- **OCTAVE → Attack Trees**: OCTAVE's threat scenarios become root goals for attack tree analysis
