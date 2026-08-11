# Contributing

Contributions are welcome. Here's how to get involved.

## Development Setup

```bash
# Clone the repo
git clone git@github.com:RedHatProductSecurity/agentic-threat-modeling.git
cd agentic-threat-modeling

# Register as a local module
lola mod add .

# Install for your AI assistant
lola install threat-model -a claude-code

# After making changes, regenerate
lola update
```

## Types of Contributions

### Adding a New Framework

1. Create a reference file at `module/skills/threat-model-analyze/reference/<framework>.md`
2. Follow the pattern of existing reference files — include: methodology, threat categories, analysis steps, when to use, comparison with other frameworks
3. Update the dispatcher skill (`module/skills/threat-model/SKILL.md`) to include the new framework in routing logic
4. Update the main command (`module/commands/threat-model.md`) to accept the new framework name
5. Update the README with the new framework

### Improving Prompts

The skill quality depends on prompt quality. When improving SKILL.md files:
- Keep each SKILL.md under 5000 tokens (lola best practice)
- Be specific and actionable in instructions
- Test with real repositories before submitting
- Include examples where they help clarify behavior

### Adding Actor Profiles

To add new threat actor personas:
1. Add the profile to `module/skills/threat-model-analyze/reference/threat-actors.md`
2. Follow the existing format: ID, Profile, Capabilities, Motivation, Typical targets, TTPs
3. Add entries to the actor selection heuristics tables
4. Update the README actor list

### Improving Output Templates

The report template (`module/skills/threat-model-report/templates/threat-model-template.md`) defines the output contract. Changes here affect all generated reports.

## Testing

Test your changes by running the module against real repositories:
1. A simple web application (REST API + database)
2. A microservices project with multiple services
3. A project with known security issues (to verify detection)

Check that:
- The correct framework is applied
- Actor suggestions make sense for the project
- Threat narratives are specific, not generic
- The report is readable by non-security engineers

## Pull Request Guidelines

- Describe what you changed and why
- Include before/after examples if you changed output format
- Test against at least one real repository
- Keep changes focused — one framework, one fix, one improvement per PR
