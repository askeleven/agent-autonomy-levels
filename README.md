# Agent Autonomy Levels

**A shared vocabulary for how much an AI agent may do without a human.**

"Autonomous AI agent" currently means nothing. One vendor's agent drafts an email you
press send on. Another's picks the recipients, writes the copy, sends it at 2am, and
files the result in your CRM. Both are sold with the same sentence.

Buyers have no words for the difference, so they cannot ask for it, contract for it, or
audit it.

This is six levels, L0 to L5, borrowed in shape from the SAE driving-automation levels
for the same reason those exist: the useful question is never "is it automated" but "who
is responsible when it is wrong, and how would anyone know."

**[Read the spec](SPEC.md)**

---

## The levels, in one line each

| | | |
|---|---|---|
| **L0** | Suggest | Generates text. Touches nothing. You act. |
| **L1** | Draft in place | Writes drafts into your systems. You commit each one. |
| **L2** | Propose and approve | Picks targets and composes. You approve each item. |
| **L3** | Bounded autonomy | Acts freely inside an envelope enforced in code. |
| **L4** | Supervised autonomy | Owns a domain. Requires working detection, not just logging. |
| **L5** | Delegated ownership | Owns an outcome. Not responsibly deployable today for customer-facing work. |

---

## Three things this spec insists on that most don't

**Autonomy is a property of a capability, not of an agent.** No agent is "an L3 agent."
It is L4 for internal notes, L2 for customer email, L0 for refunds. Every real deployment
is a matrix. A single number on a datasheet is marketing.

**Gating the send is not gating the decision.** Teams approve the wording of an outreach
email and never approve, or even see, how the two hundred recipients were chosen.
Selection is where the damage lives. We call this the selection trap and it is the most
common design error in this field.

**Reversibility beats approval.** Approval is expensive, scales badly, and degrades under
load, which is why a human reviewing twelve drafts reads them and a human reviewing two
hundred does not. Making an action undoable is cheap and does not degrade. Spend approval
only on the genuinely irreversible.

---

## Who wrote this and why you should discount it accordingly

[AskEleven](https://askeleven.com) builds and runs managed AI employees for
small businesses. So we have an obvious interest in how this vocabulary settles.

Read it with that in mind. What we will say for it is that the failure modes named in the
spec are ones we caused in production, on real customer email, and not ones we imagined
in a workshop: the stale approval that fired eleven hours late, the scheduler that ran
four hours early in the wrong timezone, the silent stall that cost a working day, the
one-agent request satisfied with a whole-company switch.

The spec is deliberately unflattering to a lot of what is currently sold as an AI
employee, including some of the ways we have built ours.

---

## Using it

Licensed **CC BY 4.0**. Quote it, fork it, put it in your RFP, ship a competing version.
Attribution appreciated, permission not required.

If you are a buyer: ask a vendor for the table in
[Stating your level honestly](SPEC.md#stating-your-level-honestly), then ask the three
questions under it. Where is the enforcement, show me a refusal, and what fired last time
it broke.

## Contributing

Disagreement is the point. See [CONTRIBUTING.md](CONTRIBUTING.md). The most valuable
issue you can open is a failure mode you have hit at L3 or above that the spec does not
describe.

## Status

**v0.1, draft for public comment.** The levels are stable enough to use in a contract.
The conformance table format is not. Reviewed and updated at least quarterly; see
[CHANGELOG.md](CHANGELOG.md).
