# PASTA (Process for Attack Simulation and Threat Analysis)

PASTA is a seven-stage, risk-centric threat modeling methodology. Unlike STRIDE's per-element classification approach, PASTA works top-down from business objectives to technical attack simulation, making it ideal when the audience cares about business impact and compliance — not just technical vulnerabilities.

## The Seven Stages

### Stage 1: Define Business Objectives
Establish the business context and risk appetite before any technical analysis.

**Questions to drive this stage:**
- What is the business purpose of this system/application?
- What are the key business processes it supports?
- What compliance or regulatory requirements apply (PCI-DSS, HIPAA, SOC2, GDPR, FedRAMP)?
- What is the organization's risk tolerance? What level of residual risk is acceptable?
- What are the business-critical SLAs (uptime, response time, data freshness)?
- What would the financial/reputational impact be if this system were compromised?

**Output:** A business impact profile that anchors all subsequent technical analysis in real-world consequences.

### Stage 2: Define Technical Scope
Decompose the system into its technical components and boundaries.

**Activities:**
- Inventory all application components, services, APIs, and infrastructure
- Identify technology stack (languages, frameworks, databases, cloud services)
- Map network topology and deployment architecture
- Document authentication and authorization mechanisms
- Identify data flows and data classification levels
- Define the system boundary — what's in scope vs. out of scope

**Output:** A technical architecture document with component inventory, technology fingerprint, and scope boundaries.

### Stage 3: Application Decomposition
Break down the application into analyzable units with trust boundaries.

**Activities:**
- Create data flow diagrams (DFDs) showing how data moves through the system
- Identify trust boundaries (where privilege or trust levels change)
- Map entry points (APIs, UI forms, file uploads, message queues, admin interfaces)
- Identify assets (data stores, credentials, cryptographic keys, configuration)
- Document user roles and permission models
- Identify third-party integrations and their trust levels

**Output:** DFDs with trust boundaries, asset inventory, and entry point catalog.

### Stage 4: Threat Analysis
Identify threats relevant to this specific system using threat intelligence.

**Activities:**
- Research known threats to the technology stack (CVE databases, vendor advisories)
- Analyze historical incidents in similar systems or the same industry
- Map relevant threat actors and their TTPs (Tactics, Techniques, Procedures)
- Use attack libraries (MITRE ATT&CK, CAPEC, OWASP) to identify applicable attack patterns
- Consider insider threats specific to the organization's context
- Identify emerging threats relevant to the deployment model (cloud, container, serverless)

**Output:** A threat library specific to this system, with threat actors, attack patterns, and historical context.

### Stage 5: Vulnerability Analysis
Map vulnerabilities in the current implementation to the identified threats.

**Activities:**
- Correlate threats from Stage 4 with actual system weaknesses
- Review code for common vulnerability patterns (OWASP Top 10, CWE Top 25)
- Analyze configuration for security misconfigurations
- Assess cryptographic implementations for weaknesses
- Evaluate dependency vulnerabilities (known CVEs in libraries)
- Check for design-level vulnerabilities (broken access control, insecure defaults)
- Review results from any available SAST/DAST/SCA scans

**Output:** A vulnerability map linking system weaknesses to specific threats.

### Stage 6: Attack Modeling and Simulation
Model realistic attack scenarios by combining threats and vulnerabilities.

**Activities:**
- Build attack trees for the highest-risk threat/vulnerability combinations
- Model kill chains: how would an attacker chain vulnerabilities to achieve their goal?
- Simulate attack paths from initial access to objective (data exfil, lateral movement, privilege escalation)
- Estimate the effort, skill, and resources required for each attack path
- Identify which attacks are blocked by existing controls and which succeed
- Prioritize by attacker ROI — which paths give the most reward for the least effort?

**Output:** Attack scenarios with kill chains, success probability, and existing control effectiveness.

### Stage 7: Risk and Impact Analysis
Quantify risk and produce actionable mitigation recommendations.

**Activities:**
- Score each attack scenario using a consistent risk framework:
  - **Likelihood** = threat capability + vulnerability exploitability + existing controls
  - **Impact** = business impact from Stage 1 + data sensitivity + blast radius
  - **Risk = Likelihood x Impact**
- Rank risks from highest to lowest
- For each risk, recommend mitigations with:
  - Expected risk reduction
  - Implementation effort (cost, time, complexity)
  - Priority (immediate / short-term / long-term)
- Calculate residual risk after proposed mitigations
- Identify accepted risks with documented rationale

**Output:** A prioritized risk register with mitigation roadmap, effort estimates, and residual risk assessment.

## When to Use PASTA

PASTA is the right choice when:
- The audience includes business stakeholders, not just engineers
- Compliance requirements drive the analysis (PCI, HIPAA, SOC2)
- You need to quantify risk in business terms (financial impact, SLA breach probability)
- The system handles regulated data or critical business processes
- You want attack simulation, not just threat enumeration

## PASTA vs. STRIDE

| Aspect | STRIDE | PASTA |
|---|---|---|
| Approach | Bottom-up, per-element | Top-down, business-driven |
| Output focus | Threat classification | Risk quantification |
| Audience | Engineers | Engineers + business stakeholders |
| Depth | Broad threat identification | Deep attack simulation |
| Effort | Moderate | High (7 full stages) |
| Best for | Design-time threat enumeration | Risk-driven security investment decisions |
