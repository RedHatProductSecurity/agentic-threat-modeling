# Discovery Interview Questions

Guided questions for building system context through user interview. Start with Phase 0 (external sources), then Phase 1 and deeper. Adapt — don't follow rigidly.

## Phase 0: External Sources
Ask before diving into system questions — external docs dramatically improve coverage.

1. **Do you have architecture documentation for this system?** Google Docs, Confluence pages, design docs, ADRs? (Provide URLs or paste content)
2. **Are there any existing threat models, security assessments, or SAR reports?** (Prior work saves time and ensures continuity)
3. **Do you have deployment or infrastructure documentation?** Terraform configs, topology diagrams, Kubernetes manifests, network diagrams? (File paths or paste)
4. **Are there machine-readable specs I should ingest?** OpenAPI/Swagger specs, SBOMs, product definition files? (File paths)

If the user has external sources, ingest them before proceeding. If not, move to Phase 1.

## Phase 1: The Big Picture
Ask these first to understand what you're dealing with.

1. **What does this system do?** What problem does it solve and for whom?
2. **Who are the users?** End users, admins, other services, third-party integrations?
3. **Where does it run?** Cloud provider, on-prem, hybrid? Single region or multi-region?
4. **What's the most important thing this system must not lose, leak, or break?** (This reveals crown jewels)

## Phase 2: Architecture and Data
Based on Phase 1, drill into the technical landscape.

5. **Walk me through what happens when a typical user request comes in** — from the moment it hits your infrastructure to the response going back. (This maps data flows naturally)
6. **What databases or data stores does the system use?** What kind of data lives in each?
7. **What external services does it talk to?** Payment processors, email providers, analytics, third-party APIs?
8. **How do users authenticate?** Passwords, SSO, OAuth, API keys, certificates?
9. **What roles or permission levels exist?** What can each role do that others can't?

## Phase 3: Security Controls
Understand what's already in place.

10. **What security measures are you most confident in?** (Reveals known strengths)
11. **What security concerns keep you up at night?** (Reveals known worries — and often the real threats)
12. **Have there been any security incidents or close calls?** (Historical context is invaluable)
13. **What compliance requirements apply?** PCI, HIPAA, SOC2, GDPR, FedRAMP, internal policies?

## Phase 4: Boundaries and Trust
Map where trust changes.

14. **What's internet-facing vs. internal-only?** Which endpoints can the public reach?
15. **Are there admin or management interfaces?** How are they accessed and protected?
16. **What third parties have access to your systems or data?** Vendors, contractors, partners?
17. **How are secrets managed?** API keys, database passwords, certificates — where do they live?
18. **Are any security-relevant defaults or gaps intentional?** e.g., relaxed auth on an internal-only service, no encryption on a boundary you consider trusted. Record the reasoning — this determines whether the analysis phase treats it as an oversight or a deliberate tradeoff.

## Phase 5: Operations and Change
Understand how the system evolves.

19. **How does code get to production?** CI/CD pipeline, manual deploys, review process?
20. **Who has production access?** How is it granted and revoked?
21. **How do you monitor for security issues?** Logging, alerting, SIEM, anomaly detection?
22. **What's the incident response process?** Who gets paged and what do they do?

## Phase 6: Threat Actor Context
Help the skill suggest relevant actors.

23. **What industry are you in?** (Determines regulatory and actor landscape)
24. **How visible is your organization?** (High-profile → hacktivist risk)
25. **Do you have competitors who might benefit from your data?**
26. **How many employees/contractors have access to sensitive systems?** (Insider risk surface)

## Adaptive Follow-ups

If the user mentions:
- **Microservices** → ask about service-to-service auth, service mesh, API gateway
- **Kubernetes** → ask about RBAC, network policies, pod security, secrets management
- **Serverless** → ask about function permissions, cold start attacks, event injection
- **Machine learning** → ask about model poisoning, training data sensitivity, inference API abuse
- **Mobile app** → ask about certificate pinning, local storage, API key embedding
- **IoT** → ask about firmware updates, physical access, communication protocols
- **Multi-tenant** → ask about tenant isolation, data segregation, noisy neighbor
