# Attack Trees

Attack trees are a structured, goal-oriented method for modeling threats. Unlike frameworks that classify threats by category (STRIDE) or process (PASTA), attack trees start with an attacker's objective and decompose it into all possible paths to achieve that goal. They are ideal for deep-dive analysis of specific high-value threats.

## Structure

An attack tree has three elements:

### Root Node (Goal)
The attacker's ultimate objective. Be specific:
- "Exfiltrate customer PII from the billing database"
- "Gain administrative access to the production Kubernetes cluster"
- "Disrupt the payment processing service for >4 hours"

Vague goals like "compromise the system" produce vague trees. Each tree should model one specific attacker objective.

### Intermediate Nodes (Sub-goals)
Steps or conditions the attacker must achieve on the path to the root goal. Each intermediate node is decomposed further until you reach leaf nodes.

### Leaf Nodes (Attack Steps)
Concrete, atomic actions the attacker takes. Each leaf should be specific enough to assess:
- **Feasibility**: Can this actually be done?
- **Cost**: What resources does the attacker need?
- **Skill**: What expertise is required?
- **Detectability**: Would this action be noticed?

## Node Relationships

### OR Nodes
The attacker needs to succeed at **any one** child to achieve the parent. Most nodes are OR — there are usually multiple paths to a goal.

```
Goal: Access admin panel
├── OR: Steal admin credentials
├── OR: Exploit authentication bypass (CVE-XXXX)
├── OR: Escalate from regular user account
└── OR: Access via exposed debug endpoint
```

### AND Nodes
The attacker must succeed at **all** children to achieve the parent. AND nodes represent multi-step attacks.

```
Goal: Exfiltrate data via supply chain
├── AND: Compromise CI/CD pipeline
│   ├── OR: Steal CI credentials from developer laptop
│   └── OR: Submit malicious PR that passes review
├── AND: Inject data exfiltration code
└── AND: Evade runtime monitoring
```

## Building an Attack Tree

### Step 1: Define the Root Goal
Choose a specific, high-value attacker objective. Good sources:
- Crown jewels from the asset inventory (what's most valuable?)
- Historical incidents in similar systems
- Compliance-driven scenarios (what breach would trigger regulatory action?)
- Input from threat actor profiling (what would this actor want?)

### Step 2: Decompose Into Sub-goals
Ask: "What must the attacker do to achieve this?" Decompose recursively until you reach concrete actions.

**Useful decomposition questions:**
- What are the prerequisites for this step?
- What alternative approaches exist?
- What must the attacker know, have, or control?
- What existing controls block this path?

### Step 3: Annotate Leaf Nodes
For each leaf node, assess:

| Attribute | Scale | Description |
|---|---|---|
| Cost | Low / Medium / High | Resources the attacker needs (time, money, compute) |
| Skill | Low / Medium / High | Expertise required (script kiddie → APT) |
| Access | Remote / Local / Physical / Insider | What access level is needed |
| Detectability | High / Medium / Low | Likelihood of detection by existing controls |
| Feasibility | Proven / Theoretical / Speculative | Is this attack known to work? |

### Step 4: Calculate Path Risk
For each path from root to leaves:
- **OR nodes**: the path risk is the risk of the easiest child (attacker picks the cheapest path)
- **AND nodes**: the path risk is limited by the hardest child (the chain is as strong as its weakest link — but cost accumulates)

The highest-risk path is the one that is feasible, low-cost, requires low skill, and has low detectability.

### Step 5: Identify Mitigations
Focus mitigations on:
1. **Chokepoints**: nodes that appear on many attack paths (blocking one node blocks many paths)
2. **Cheapest paths**: the paths an attacker would actually take
3. **AND node dependencies**: breaking any one AND child blocks the entire path

## Example: Attack Tree for "Steal API Keys"

```
ROOT: Obtain production API keys
├── OR: Extract from source code
│   ├── OR: Search git history for committed secrets
│   ├── OR: Find hardcoded keys in config files
│   └── OR: Access .env files via path traversal
├── OR: Extract from running environment
│   ├── OR: Read environment variables via SSRF
│   ├── OR: Dump process memory
│   └── OR: Access container metadata endpoint
├── OR: Intercept in transit
│   ├── AND: Position for MITM
│   │   ├── AND: Compromise network (ARP spoof / DNS hijack)
│   │   └── AND: Downgrade or strip TLS
│   └── OR: Access CI/CD logs where keys are printed
├── OR: Obtain from key management system
│   ├── OR: Exploit vault/KMS misconfiguration
│   └── OR: Compromise service account with vault access
└── OR: Social engineering
    ├── OR: Phish developer with access
    └── OR: Bribe/coerce insider
```

## When to Use Attack Trees

- Deep analysis of a specific, high-value threat scenario
- When you need to understand all possible paths to a specific attacker goal
- Communicating attack complexity to non-technical stakeholders (trees are visual and intuitive)
- Prioritizing defensive investments by identifying chokepoints
- Red team / purple team exercise planning
- When other frameworks identified a critical threat and you need to explore it in depth

## Attack Trees + Other Frameworks

Attack trees pair well with other frameworks:
- **STRIDE → Attack Trees**: STRIDE identifies threat categories; build attack trees for the highest-risk threats
- **PASTA Stage 6**: PASTA's attack modeling stage naturally produces attack trees
- **Threat Actor Profiling → Attack Trees**: Model what a specific actor persona would do by pruning branches that exceed their capability/motivation

## Output Format

Each attack tree should include:
1. **Root goal statement** with context (what asset, what impact)
2. **Tree diagram** (text-based, using indentation and OR/AND labels)
3. **Leaf annotations** (cost, skill, access, detectability, feasibility)
4. **Highest-risk paths** highlighted with rationale
5. **Recommended mitigations** mapped to chokepoint nodes
6. **Residual risk** after proposed mitigations
