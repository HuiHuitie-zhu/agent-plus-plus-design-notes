# Publishing Notes

> Use this checklist before copying any DeepShen note into a public GitHub repository, blog, or social post.

## Publishable

- General design principles.
- Abstract diagrams and pseudocode.
- Sanitized JSON schema examples.
- Failure modes without private transcripts.
- Lessons learned from prototype behavior.
- High-level module boundaries.

## Do Not Publish

- Private conversations or message screenshots.
- Full persona prompts or relationship-specific wording.
- User profile content.
- Real device names, local paths, hostnames, IP addresses, or deployment topology.
- API keys, bot tokens, environment variables, config files, or launch scripts.
- Raw logs, diary entries, or extracted long-term memory.
- Any implementation detail that makes the private deployment directly reproducible.

## Sanitization Rules

Before publishing, check that the note:

1. Uses `user` instead of any real person name.
2. Uses generic system names instead of personal bot names when possible.
3. Shows toy examples, not real chat snippets.
4. Keeps JSON examples schema-level and fake.
5. Avoids local filesystem paths.
6. Avoids channel-specific details unless they are generic.
7. Frames claims as design lessons, not proof that an AI has human emotions.

## Recommended Public Positioning

Use:

```text
stateful companion AI
external emotional state
expression guardrails
reply-first runtime
layered memory
```

Avoid:

```text
real emotions
AI lover
private relationship simulation
full companion clone
```

## Final Review Query

Run a text search for:

```text
Hui|辉辉|阿深|真实对话|私人|API key|token|/Users|localhost|192.168|Telegram|Feishu|飞书|Mac mini|SOUL|USER
```

Any hit must be manually reviewed before publication. Some hits in this checklist are expected; hits in article bodies need stronger scrutiny.
