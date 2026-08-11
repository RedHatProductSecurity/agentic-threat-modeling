# Threat Actor Personas

This reference defines threat actor profiles used during threat analysis. The skill automatically suggests likely actors based on system context, and users can request analysis from specific actor perspectives.

## Actor Profiles

### Nation-State / Advanced Persistent Threat (APT)
**ID:** `apt`

**Profile:** State-sponsored or state-affiliated groups with virtually unlimited resources, patience, and sophistication. They operate covertly over months or years, targeting specific organizations or sectors for strategic advantage.

**Capabilities:**
- Zero-day exploit development and acquisition
- Custom malware and implant development
- Long-term persistent access (months to years)
- Advanced social engineering and recruitment of insiders
- Supply chain compromise at scale
- Signals intelligence and network interception capabilities

**Motivation:** Espionage (political, military, economic), strategic disruption, pre-positioning for future conflict, intellectual property theft, surveillance.

**Typical targets:** Government agencies, defense contractors, critical infrastructure (energy, telecom, water), research institutions, technology companies with valuable IP, political organizations, diplomatic entities.

**TTPs (MITRE ATT&CK alignment):** Initial access via spear-phishing, supply chain compromise, or zero-day exploitation. Lateral movement via credential theft and living-off-the-land. Persistence via rootkits, firmware implants, or compromised update mechanisms. Exfiltration via covert channels, steganography, or compromised legitimate services.

---

### Organized Crime
**ID:** `crime`

**Profile:** Financially motivated criminal organizations operating as businesses — with developers, infrastructure, customer support (for ransomware), and money laundering operations. Increasingly sophisticated and well-resourced.

**Capabilities:**
- Ransomware-as-a-Service (RaaS) platforms
- Large-scale phishing and credential harvesting
- Botnet operation (DDoS-for-hire, credential stuffing)
- Dark web data marketplace operation
- Money laundering via cryptocurrency
- Purchase of zero-days and exploit kits

**Motivation:** Financial gain — ransomware payments, data sale, fraud, business email compromise, cryptojacking.

**Typical targets:** Organizations with valuable data (PII, payment card data, health records), organizations likely to pay ransoms (hospitals, municipalities, law firms), e-commerce and financial services, any organization with weak perimeter security.

**TTPs:** Initial access via phishing, RDP brute force, or exploiting public-facing applications. Rapid lateral movement and privilege escalation. Data exfiltration before ransomware deployment (double extortion). Use of commodity malware and affiliate programs.

---

### Malicious Insider
**ID:** `insider`

**Profile:** Current or former employees, contractors, or business partners who intentionally misuse their authorized access. They have domain knowledge, understand internal processes, and know where valuable assets are.

**Capabilities:**
- Legitimate access to systems and data
- Knowledge of internal processes, security controls, and their gaps
- Ability to bypass controls designed for external threats
- Social trust — actions may not be questioned
- Access to physical facilities

**Motivation:** Financial gain (selling data, fraud), revenge (disgruntled employee), ideology (whistleblowing gone wrong), coercion (blackmailed by external actor), career advancement (stealing IP for a new employer).

**Typical targets:** Sensitive data they have access to (customer data, trade secrets, financial data), admin/root accounts, security and monitoring systems (to cover tracks), source code and intellectual property.

**TTPs:** Data exfiltration via authorized channels (email, cloud storage, USB), privilege abuse, creating backdoor accounts, modifying or disabling security controls, accessing data outside normal job function.

---

### Negligent Insider
**ID:** `negligent`

**Profile:** Employees or contractors who cause security incidents through carelessness, lack of training, or process failures — without malicious intent. This is the most common "threat actor" by incident volume.

**Capabilities:** Same access as a malicious insider, but the "attack" is accidental.

**Motivation:** None — incidents are unintentional. Caused by: rushing, lack of security awareness, confusing interfaces, poor processes, shadow IT adoption.

**Common scenarios:**
- Falling for phishing/social engineering attacks
- Misconfiguring cloud storage (public S3 buckets, open databases)
- Committing secrets to public repositories
- Sending sensitive data to wrong recipients
- Using weak or reused passwords
- Connecting to unsecured networks
- Installing unauthorized software
- Failing to apply patches or updates

**Typical targets:** Anything accessible — the threat is accidental exposure, not targeted theft.

**TTPs:** Not intentional — these are process failures. Most common: clicking phishing links, misconfiguring systems, mishandling data.

---

### Hacktivist
**ID:** `hacktivist`

**Profile:** Ideologically motivated individuals or groups who use hacking to make political or social statements. They seek maximum public visibility for their actions.

**Capabilities:**
- DDoS attacks (using botnets or DDoS-for-hire)
- Website defacement
- Data leaks for embarrassment or activism
- Doxing of employees or executives
- Social media account takeover
- Moderate technical skill (varies widely within groups)

**Motivation:** Ideological — social justice, political activism, environmental causes, anti-corporate sentiment, retaliation for perceived wrongdoing. Success is measured in media attention and public impact.

**Typical targets:** Organizations with controversial practices or public visibility — governments, large corporations, military/defense, fossil fuel companies, organizations involved in social controversies.

**TTPs:** DDoS, web application attacks (SQLi, XSS for defacement), credential leaks, social engineering, exploiting known vulnerabilities in public-facing systems. Favor impact and visibility over stealth.

---

### Opportunistic Attacker
**ID:** `opportunistic`

**Profile:** Low-to-medium skill attackers using automated tools, known exploits, and spray-and-pray tactics. They don't target specific organizations — they scan for easy victims.

**Capabilities:**
- Automated vulnerability scanning (Shodan, Masscan, Nuclei)
- Known exploit deployment (Metasploit, public PoCs)
- Credential stuffing with leaked databases
- Script kiddie tools and exploit kits
- Basic social engineering

**Motivation:** Low-effort financial gain (cryptomining, spam relays, data for sale), bragging rights, skill development.

**Typical targets:** Any system with unpatched vulnerabilities, default credentials, exposed admin interfaces, misconfigured cloud resources, or missing basic security controls. They go for volume, not value.

**TTPs:** Scanning internet-facing services for known CVEs, brute-forcing common credentials, exploiting default configurations, deploying cryptocurrency miners, pivoting from compromised systems to find more valuable targets.

---

### Competitor
**ID:** `competitor`

**Profile:** Business rivals seeking competitive advantage through unauthorized access to trade secrets, pricing strategies, customer data, or strategic plans. May act directly or hire intermediaries.

**Capabilities:**
- Hiring skilled hackers or private intelligence firms
- Recruiting insiders at target organizations
- Legal-but-aggressive competitive intelligence (OSINT, social engineering)
- Corporate espionage tradecraft

**Motivation:** Business advantage — pricing intelligence, customer lists, product roadmaps, merger/acquisition plans, patent-pending research, bidding strategies.

**Typical targets:** Trade secrets, R&D data, pricing models, customer databases, strategic plans, partnership agreements, patent applications.

**TTPs:** Insider recruitment, targeted phishing of key personnel, legal discovery abuse, supply chain infiltration, attending conferences to social-engineer employees, dumpster diving, OSINT aggregation.

---

### Supply Chain Attacker
**ID:** `supply-chain`

**Profile:** Attackers who compromise upstream dependencies, tools, or services to reach downstream targets. This may be any other actor type using the supply chain as an attack vector.

**Capabilities:**
- Compromising open-source package maintainers or registries
- Injecting malicious code into build systems or CI/CD pipelines
- Compromising commercial software vendors (SolarWinds-style)
- Typosquatting package names
- Dependency confusion attacks

**Motivation:** Varies by underlying actor — nation-states use supply chain for broad access, criminals for payload distribution, competitors for targeted sabotage.

**Typical targets:** Build systems, package registries (npm, PyPI, Maven), CI/CD pipelines, software update mechanisms, managed service providers, commercial off-the-shelf software.

**TTPs:** Malicious package publication, account takeover of maintainers, build system compromise, update mechanism hijacking, dependency confusion, typosquatting.

## Actor Selection Heuristics

The skill automatically suggests relevant actors by analyzing system context:

### By Data Stored
| Data Type | Likely Actors |
|---|---|
| PII (names, emails, SSNs) | Organized crime, insider, negligent |
| Payment/financial data | Organized crime, opportunistic |
| Health records (PHI) | Organized crime, insider |
| Trade secrets / IP | Competitor, APT, insider |
| Government/military data | APT, hacktivist |
| Authentication credentials | All actors |

### By System Exposure
| Exposure | Likely Actors |
|---|---|
| Internet-facing web app | Opportunistic, hacktivist, organized crime |
| Internal-only tool | Insider (malicious + negligent) |
| API for third-party integration | Supply chain, opportunistic |
| Mobile application | Opportunistic, organized crime |
| IoT / embedded system | APT, opportunistic |
| Critical infrastructure | APT, hacktivist |

### By Technology Stack
| Technology | Likely Actors |
|---|---|
| Heavy open-source dependencies | Supply chain |
| Cloud-native (AWS/GCP/Azure) | Negligent (misconfiguration), opportunistic |
| Legacy systems | Opportunistic (unpatched CVEs) |
| CI/CD with external runners | Supply chain |
| Containerized / Kubernetes | Opportunistic (misconfiguration), APT |

### By Compliance Context
| Regulatory Context | Likely Actors |
|---|---|
| HIPAA (healthcare) | Organized crime, insider |
| PCI-DSS (payment) | Organized crime, opportunistic |
| ITAR/EAR (export control) | APT |
| SOX (financial) | Insider, organized crime |
| GDPR/CCPA (privacy) | Organized crime, negligent |
| FedRAMP/FISMA (government) | APT, hacktivist |

## Using Actor Personas in Analysis

When applying any framework with actor personas:

1. **Default behavior**: The skill analyzes system context and suggests 2-4 most likely actors with rationale.
2. **Per-threat mapping**: Each identified threat includes which actors would exploit it and their specific motivation.
3. **Actor-focused view**: Users can request the threat model from a specific actor's perspective — "show me this system through the eyes of a supply chain attacker."
4. **Kill chain narratives**: For high-risk threats, the report includes a narrative of how a specific actor would carry out the attack from initial reconnaissance to objective completion.
