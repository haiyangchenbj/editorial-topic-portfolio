## Description:

Evaluates a portfolio of technology, AI, data, cloud, or enterprise-software content topics by normalizing inputs, applying hard gates, scoring eligible topics, selecting one primary topic and two backups, and preparing a confirmed Notion writeback preview.

This skill is ready for commercial/non-commercial use.

## Publisher:

[haiyangchenbj](https://clawhub.ai/user/haiyangchenbj)

### License/Terms of Use:

MIT-0

## Use Case:

Editorial strategists, analysts, and content operators use this skill to evaluate candidate technology topics, choose the current writing portfolio, and prepare reviewed changes for a Notion-backed topic database.

### Deployment Geography for Use:

Global

## Known Risks and Mitigations:

Risk: A Notion integration could modify the wrong topic records or fields if writeback is not reviewed.

Mitigation: Keep writeback in preview or dry-run until the portfolio, target database, page count, fields, and actions are explicitly approved; use page IDs rather than title matching.

Risk: Credentials, database IDs, mappings, and private positioning data could be exposed if stored in reusable skill files or logs.

Mitigation: Keep tokens and mappings outside the skill directory, scope the Notion integration to the topic database, and avoid logging token values or private notes.

Risk: Topic recommendations can be misleading if facts, dates, policy status, product status, or evidence are stale or unverifiable.

Mitigation: Apply the fact, timeliness, originality, argument-capacity, positioning, and privacy gates before scoring; block or hold topics with unresolved evidence gaps.

## Reference(s):

- [ClawHub Skill Page](https://clawhub.ai/haiyangchenbj/skills/editorial-topic-portfolio-skill)
- [Evaluation Gates](references/evaluation-gates.md)
- [Scoring Model](references/scoring-model.md)
- [Portfolio Rules](references/portfolio-rules.md)
- [Notion Adapter Interface](references/notion-adapter-interface.md)
- [Replay Evaluation](references/replay-evaluation.md)

## Skill Output:

**Output Type(s):** [text, markdown, JSON, shell commands, configuration, guidance]

**Output Format:** [Markdown reports, JSON records, shell command snippets, and Notion change previews]

**Output Parameters:** [1D]

**Other Properties Related to Output:** [Outputs include gate results, scores, portfolio decisions, change previews, validation records, writeback results, and readback verification.]

## Skill Version(s):

1.0.7 (source: server evidence and frontmatter)

## Ethical Considerations:

Users should evaluate whether this skill is appropriate for their environment, review any generated or modified files before relying on them, and apply their organization's safety, security, and compliance requirements before deployment.
