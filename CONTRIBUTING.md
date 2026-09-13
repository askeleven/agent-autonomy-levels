# Contributing

This spec is trying to become shared vocabulary. That only works if people outside
AskEleven shape it.

## What is most useful

**Failure modes we missed.** If you have run an agent at L3 or above and hit something
the spec does not describe, that is the highest-value issue you can open. Concrete
incidents beat theory. What was allowed, what happened, what would have caught it.

**Places the levels do not fit your domain.** The spec is written from experience with
customer communication, CRM, and scheduling. If you are running agents against code,
infrastructure, finance, or clinical workflows and the ladder breaks down, say where.

**Ambiguity.** If two reasonable people would classify the same capability differently,
the level definition is underspecified. Show us the case.

**Disagreement with our positions.** The spec takes stances: that L5 is not responsibly
deployable for customer-facing work, that reversibility beats approval, that a
prompt-based envelope is not an envelope. Argue with any of them.

## What is less useful

Wording polish on its own, and requests to add levels. Six is already one more than most
people will use.

## How

Open an issue first for anything substantive, so the discussion is public before the
diff. Pull requests are welcome for typos, broken links, and agreed changes.

## Ground rules

- No vendor pitches in issues, ours included.
- Anonymize incidents. Never name a customer or paste real recipient data.
- The spec is CC BY 4.0. By contributing you agree your contribution ships under that
  license.

## Maintenance

Maintained by AskEleven. Reviewed at least quarterly, and after any incident that teaches
us something. Issues get a response within a week; if one goes stale, bump it.
