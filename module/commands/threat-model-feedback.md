---
description: Share feedback on a generated threat model by filing a GitHub issue.
argument-hint: ""
---

Collect feedback about the most recent threat model and file it as a GitHub issue using the [Feedback issue template](https://github.com/RedHatProductSecurity/agentic-threat-modeling/blob/main/.github/ISSUE_TEMPLATE/feedback.yml).

Ask the user these questions one at a time:

1. **How useful was this threat model?**
   Options: Very useful / Somewhat useful / Not useful

2. **Which section was most valuable?**
   Options: Executive summary / Architecture overview / Threats / Attack scenarios / Mitigations / Data classification / Other

3. **Was anything missing or wrong?** (free text, or "nothing")

4. **Were there any false positives — threats that don't actually apply?** (free text, or "none")

5. **On a scale of 1-5, how would you rate the overall quality?** (1 = poor, 5 = excellent)

6. **Would you use this tool again?**
   Options: Yes / Maybe / No

7. **Any additional comments?** (free text, or skip)

## Filing the Issue

After collecting answers, try these in order — a filed issue is more useful than a link the user might not click, so only fall back to a link if neither tool is available.

**1. GitHub CLI (`gh`), if installed and authenticated:**
Check with `gh auth status`. If authenticated, create the issue directly:
```bash
gh issue create \
  --repo RedHatProductSecurity/agentic-threat-modeling \
  --title "[Feedback] <one-line summary from the usefulness + rating answers>" \
  --label feedback \
  --body "$(cat <<'EOF'
**How useful was this threat model?** {usefulness}

**Which section was most valuable?** {most_valuable}

**Was anything missing or wrong?** {missing}

**Were there any false positives?** {false_positives}

**Overall quality rating (1-5):** {rating}

**Would you use this tool again?** {use_again}

**Additional comments:** {comments}
EOF
)"
```
Share the issue URL that `gh issue create` returns.

**2. A connected GitHub MCP server, if `gh` isn't installed or isn't authenticated:**
If an MCP tool for creating GitHub issues is available in this session, use it against the same repository (`RedHatProductSecurity/agentic-threat-modeling`) with the same title, labels, and body as above.

**3. Fallback: a pre-filled link, if neither of the above is available:**
Construct a pre-filled "new issue" URL against the Feedback issue form, URL-encoding each answer into its field:
```
https://github.com/RedHatProductSecurity/agentic-threat-modeling/issues/new?template=feedback.yml&title=%5BFeedback%5D&usefulness={usefulness}&most-valuable={most_valuable}&missing={missing}&false-positives={false_positives}&rating={rating}&use-again={use_again}&comments={comments}
```

Present it as:

> **Thanks for the feedback!** Click here to file it: [Submit Feedback]({constructed_url})
>
> The issue will open pre-filled with your answers on GitHub. Just review and click "Submit new issue."
