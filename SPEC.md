# Agent Autonomy Levels

**Version 0.1 (draft for public comment)**

A shared vocabulary for describing how much an AI agent is allowed to do without a human
in the loop.

---

## Why this exists

"Autonomous AI agent" currently means nothing. Two vendors use the same phrase for
products that differ by a factor of a thousand in blast radius. One drafts an email you
press send on. The other picks the recipients, writes the copy, sends it, and files the
result in your CRM at 2am. Both call it an AI employee.

Buyers have no words for the difference, so they cannot ask for it, contract for it, or
audit it. Operators have no words for it either, so they discover the difference during
an incident.

This document defines six levels, L0 to L5. It borrows its shape from the SAE levels for
driving automation, for the same reason those exist: the interesting question is never
"is it automated" but "who is responsible when it is wrong, and how would anyone know."

It is written from production experience running agents that send real email to real
customers on behalf of real businesses. The failure modes named here are ones we have
caused.

---

## The one rule that matters

**Autonomy is a property of a capability, not of an agent.**

An agent is not "an L3 agent." An agent is L4 for writing internal CRM notes, L2 for
sending email to existing customers, and L0 for anything involving a refund. Every real
deployment is a matrix, not a number.

Any vendor, contract, or datasheet that assigns a single level to a whole agent is
describing marketing, not architecture. Ask for the matrix.

---

## The levels

Each level below states: what the agent may do, what a human must do, what must be
logged, what must be reversible, and the characteristic way it fails.

---

### L0 - Suggest

**Agent may:** generate content in a workspace the human owns. Answer questions. Draft
text on request.

**Agent may not:** touch any system of record. Produce any side effect outside the chat
surface.

**Human must:** do everything. Copy, paste, decide, act.

**Logging:** conversation history.

**Reversibility:** total. Nothing happened.

**Example:** you paste a customer complaint into a chat window and ask for three possible
replies. You pick one and type it yourself.

**Characteristic failure:** none that reaches a customer. The risk at L0 is entirely that
the human trusts a wrong answer. That is a real risk, but it is a human-judgment risk,
not an autonomy risk.

> Most "AI employees" sold in 2026 are L0 with a nicer wrapper.

---

### L1 - Draft in place

**Agent may:** write into the business's systems in a state that has no effect. Gmail
drafts. Unsent CRM notes. A calendar hold nobody has been invited to.

**Agent may not:** cause anything to leave the building or become visible to a customer.

**Human must:** perform the committing action themselves, in the native tool, one item at
a time.

**Logging:** every artifact created, with the input that produced it.

**Reversibility:** delete the draft.

**Example:** every morning the agent leaves twelve drafted replies in the shared inbox.
The owner reads them over coffee and sends the nine that are right.

**Characteristic failure:** volume fatigue. At twelve drafts a human reads each one. At
two hundred they start sending without reading, and you are running L3 while believing
you are running L1. **The level you are actually at is the level the human is actually
reviewing at.**

---

### L2 - Propose and approve

**Agent may:** select what to act on, compose the action, and queue it. Present it to a
human as a decision with a yes and a no.

**Agent may not:** commit anything without a recorded human approval of that specific
item.

**Human must:** approve or reject each item. Approval must be an affirmative act, not a
timeout.

**Logging:** the proposal, the approver's identity, the decision, the timestamp, and any
edits the human made before approving. Edits are the highest-value training signal you
will ever collect; capture them.

**Reversibility:** rejection costs nothing. Approved actions inherit the reversibility of
the underlying action.

**Example:** the agent identifies eight prospects who went quiet, writes a follow-up for
each, and puts eight cards in an approval queue. The owner edits two, approves six,
rejects two.

**Characteristic failures, both of which we have shipped:**

1. **The selection trap.** See below. Approving the message is not approving the choice
   of recipient, and the choice of recipient is usually where the damage is.
2. **Perishable approval.** An approval card that sits in a queue for eleven hours and
   then dispatches is not an approved action, it is a stale one. The world moved. The
   customer already replied, already paid, already churned. **Every approval must carry
   an expiry after which it dies rather than fires.** We learned this by sending
   duplicate emails to customers who had already been handled.

---

### L3 - Bounded autonomy

**Agent may:** act without per-item approval, inside an envelope that is written down
before the fact and enforced in code.

The envelope must enumerate, at minimum:

- **Which actions.** Send email, yes. Issue refund, no.
- **Which records.** Existing customers in this pipeline stage, yes. Anyone else, no.
- **Which channels.** Email, yes. SMS and voice, no.
- **What volume.** Per hour and per day, with a hard stop, not a warning.
- **What hours.** In whose timezone. Stated explicitly, because a scheduler that
  evaluates in UTC while the config claims local time will fire four hours early every
  day and nobody will notice for a month.

**Agent may not:** act outside the envelope. Anything outside falls back to L2, it does
not fail silently and it does not proceed.

**Human must:** define the envelope, review in aggregate on a stated cadence, and be
reachable. Not approve individual items.

**Logging:** every action, plus every envelope refusal. The refusals are the more
important log. An agent that never hits its bounds does not have bounds, it has
decoration.

**Reversibility:** required per action class before that class enters the envelope. If
you cannot undo it, it does not go in the envelope. It stays at L2.

**Example:** between 8am and 5pm Eastern on weekdays, the agent may send up to forty
follow-ups to contacts that have been in "proposal sent" for more than five business
days, using approved templates, and must stop and ask about anything that has received a
human reply.

**Characteristic failure:** envelope drift. Someone widens a bound to unblock a bad
afternoon and never narrows it back. Envelopes need dated review, and every widening
needs an owner and a reason in the log.

---

### L4 - Supervised autonomy

**Agent may:** operate across a domain, choose its own targets and sequencing within
policy, and adapt its approach between runs.

**Agent may not:** operate without working detection. This is the load-bearing
requirement of L4 and the one everyone skips.

**Human must:** review outcomes in aggregate, not actions individually. And critically:
be **alerted on failure before the customer notices.** L4 without alerting is not L4, it
is an unmonitored L5 that has not had its incident yet.

Minimum detection for L4:

- Liveness. Is it running at all? A silent agent looks identical to a well-behaved one.
- Volume anomaly, in both directions. A sudden zero is as alarming as a sudden spike.
- Error and refusal rates, trending.
- Downstream reality checks. Bounces, complaints, opt-outs, negative replies. Something
  must read the bounces. If nothing reads the bounces you will find out about your
  sending problem from your domain reputation.

**Logging:** everything at L3, plus retention long enough to reconstruct a week you were
not watching.

**Reversibility:** a kill switch that a non-technical person can reach from a phone, that
stops in-flight work rather than only preventing new work. Test it on a schedule. An
untested kill switch is a belief, not a control.

**Example:** the agent owns first-touch qualification for inbound leads. It decides who
to contact, when, how often, and through which channel, within policy. A human reads a
weekly rollup and an on-call alert fires if replies drop, bounces rise, or it goes quiet.

**Characteristic failure:** silent stall. The most common L4 incident is not the agent
doing something wrong, it is the agent doing nothing at all while everyone assumes it is
working. We have lost most of a business day to exactly this. Alert on absence.

---

### L5 - Delegated ownership

**Agent may:** own an outcome. Change its own methods, its own schedule, and its own
targets in service of a stated goal.

**Human must:** set the goal and the constraints, and intervene by exception.

**Logging:** decision rationale, not just actions. At L5 the interesting artifact is why
it changed approach, because that is the only thing a human review can act on.

**Reversibility:** full audit and rollback of a time window, not just an action.

**Honest position:** we do not believe L5 is responsibly deployable today for anything
that touches a customer, money, or a legal obligation. We include it so the ladder has a
top, and so that "we are not doing that" is a statement someone can make precisely.

Treat any vendor claiming L5 over customer communications as claiming L4 without the
monitoring.

---

## The selection trap

The single most common design error in agent governance:

> Teams gate the **send** and leave the **selection** ungated.

A human approves the wording of an outreach email. The human does not approve, and often
cannot see, how those two hundred recipients were chosen. The agent scraped a list,
inferred a fit score, and picked. The approval queue created a complete and false sense
of control, because the reviewable artifact was the least consequential half of the
decision.

Selection is where the damage lives. Wrong recipient, wrong list, wrong segment, wrong
person at a company you are already in a contract dispute with. The prose was fine.

**A capability is only at L2 if the human approves the target set as well as the
content.** If the agent picks and the human only edits copy, you are at L3 with a
cosmetic approval step, and you should describe yourself that way.

The fix is structural: split the workflow so that discovery produces a proposal that is
itself approved, before composition happens at all. Propose, ask, then commit. Two gates,
not one.

---

## Blast radius

Every control must state its scope. There are three, and conflating them is how one
employee's new rule silently changes another's behavior:

| Scope | Changes | Example |
|---|---|---|
| **Per capability** | One agent, one action class | This agent's outbound email requires approval |
| **Per tenant** | Every agent in one business | This company allows no destructive actions |
| **Per platform** | Every business on the system | Global model or provider defaults |

The failure: a request to restrain **one** agent gets satisfied with a **tenant-wide**
switch because that switch already exists and the per-agent one does not. It works. It
also silently changes the behavior of every other agent in that business, and nobody
finds out until an unrelated one misbehaves weeks later.

**If the per-capability mechanism does not exist, build it. Do not borrow the wider
switch.** A control whose scope is wider than the intent is a latent incident.

---

## Reversibility beats approval

Approval is expensive, scales badly, and degrades under load. Reversibility is cheap,
scales perfectly, and does not degrade.

Given a choice between adding an approval step and making an action undoable, make it
undoable. Reserve approval for the genuinely irreversible.

The ladder, best to worst:

1. **Undoable in place.** The CRM note can be deleted. Nobody outside saw it.
2. **Correctable.** The email went, but a follow-up fully repairs it.
3. **Visible but not correctable.** A customer saw something wrong. You can apologize.
4. **Irreversible.** Money moved. Data was deleted. A legal notice was served.

Levels 1 and 2 belong in an L3 envelope. Level 3 needs L2 approval. Level 4 needs a
human to perform the action, which is to say L1, regardless of how good the agent is.

---

## Disclosure

Autonomy level and disclosure obligation are independent axes, and both matter.

An L1 agent that drafts a message a human sends under their own name generally triggers
no disclosure obligation. An L3 agent that sends under its own identity may, depending on
jurisdiction, channel, and whether the recipient is a consumer.

Do not infer one from the other. Jurisdiction-specific requirements are tracked
separately in this repository under `disclosure/`.

---

## Stating your level honestly

A conformance claim under this spec is a table, one row per capability:

| Capability | Level | Envelope | Reversibility | Detection |
|---|---|---|---|---|
| Draft inbox replies | L2 | Existing contacts only | Rejection is free | Queue age alert |
| Internal CRM notes | L4 | Own pipeline records | Delete | Volume + liveness |
| Outbound to new prospects | L2 both gates | 40/day, business hours ET | Correctable | Bounce + complaint |
| Refunds | L0 | n/a | Irreversible | n/a |

Three tests for whether a claim is real:

1. **Where is the enforcement?** If the envelope lives in a prompt rather than in code,
   there is no envelope. Models do not reliably obey instructions under adversarial or
   unusual input, and "we told it not to" is not a control.
2. **Show me a refusal.** A log of the agent hitting a bound and stopping. No refusals
   ever recorded means the bounds are not wired to anything.
3. **What fired last time it broke?** If the answer is "the client called us," the
   detection requirement for L3 and above is not met, whatever the datasheet says.

---

## Non-goals

This spec does not address model quality, factual accuracy, bias, data residency, or
security. Those are real and orthogonal. An L1 agent can be dangerously wrong; an L4
agent can be scrupulously accurate. This document is only about how far an action travels
before a human is involved.

---

## Status and contributions

Version 0.1, published for public comment. The levels are stable enough to use in a
contract; the conformance table format is not yet.

Disagreement is the point. Open an issue, particularly if you have run something at L3 or
above and hit a failure mode not described here.

Licensed CC BY 4.0. Use it, quote it, put it in your RFP, adapt it. Attribution
appreciated, permission not required.
