# Install Booking Status Signals — Tradeoffs Register

Companion to `Install_Booking_Status_Signals_PRD.md` (v0.2, signed off 28 Jul 2026). **Not part of the PRD.**

This is the record of every decision point where options were presented and the PM chose. It exists so that when someone later asks "why is delivery best-effort?" or "why isn't technician reassignment in here?", the answer is on file instead of needing archaeology.

| # | Decision point | Chosen | Rejected options | Why (PM's stated reason) | Date |
|---|---|---|---|---|---|
| 1 | How much of "the CSP world gives up" to cover | Signal every install-journey death; the customer backend's 14-day timer becomes a fallback only | Cover only the install-retry exhaustion path (P78 → T12) and leave the rest on the old timer | "14 day fallback should only act as a fallback" | 27 Jul 2026 |
| 2 | Warehouse device return | Out of scope | Treat it as a death worth signalling | "warehouse device return should not be included as a valid signal" | 27 Jul 2026 |
| 3 | Whether every cancellation signals, or only ones that don't immediately recover | Signal **every** cancellation, even when a replacement CSP is found in the same instant | Signal only on non-recovery, after a settle window; or split by whether a promise had already been sent to the customer | "Every cancel will trigger something on the customer app. Experience is not part of this PRD — handled separately on the customer side" | 27 Jul 2026 |
| 4 | Delivery guarantee for a signal a refund depends on | Best effort, matching the existing signals on this channel | Guaranteed delivery (durable, retried, dead-lettered); or split — best-effort for cancellations, guaranteed for closures | "Tech call, not mine. My suggestion with option 1." Challenged once on the grounds that a lost closure is silently unrefunded money; PM held the position. Consequence written into the PRD as R5/G4: the 14-day fallback can no longer be removed | 27 Jul 2026 |
| 5 | How much the signal must carry | Full attribution — trigger, actor class, outgoing CSP, whether attempts remain | A minimal payload leaving the backend to infer | "All the attributes should be sent to the customer world with proper distinction so that the customer backend can build on different signals if need be" | 27 Jul 2026 |
| 6 | Two events falling due at the same instant (last attempt fails and exhausts) | Two separate signals — cancellation, then closure | One closure only; or one merged signal carrying both facts | PM chose directly | 27 Jul 2026 |
| 7 | Technician reassignment (original requirement 3) | Dropped from the PRD entirely; recorded in the Boundary as already working, protected by AC-REG-2 | Spec it as new work; or fix the blank-overwrite defect within this PRD | Verified in code that every technician change already re-sends name and mobile. "Remove this requirement altogether from this PRD if it's already happening." Blank-overwrite defect explicitly deferred to a separate ticket | 27 Jul 2026 |
| 8 | Wiom-side UI | None | Build partner or ops screens for these signals | "Yes, no UI change" — the customer team owns everything the customer sees | 27 Jul 2026 |
| 9 | Closure uniqueness | At most one per booking, keyed on `customer_id` | Allow duplicates and let the backend defend itself | "At most 1 per booking (customer_id)" | 27 Jul 2026 |
| 10 | Success metrics | Reduction in status-chasing calls, plus signal coverage | Coverage alone | Status updates are completely missing from the customer app today, so customers call repeatedly. No baseline exists — call-reason data would be needed | 27 Jul 2026 |
| 11 | Guardrails that were really rules | Removed "one closure only" and "the fallback stays"; added **Nothing existing breaks** | Keep all five as guardrails | "Guardrails you defined look like feature descriptions rather than guardrails." R2b and R5 already carried the two removed obligations | 28 Jul 2026 |
| 12 | Guardrail numbering after two removals | Renumber to a clean G1–G4 | Keep G1/G3/G5/G6 with the retired IDs unused, so an earlier draft can't be misread | "renumber to G1–G4" | 28 Jul 2026 |
| 13 | Configurability section | No configurable parameters at all | Keep signal latency, the fallback timer and the one-closure limit as C-ids | "No configurability is required here." Latency is engineering's, the fallback belongs to the customer backend, and the one-closure limit is an invariant not a setting | 28 Jul 2026 |
| 14 | How specific to be about cancellation triggers | Name all four — declined by CSP, P41 timeout, installation reported failed by CSP, P74 timeout | Keep the PRD mechanism-free with a paraphrase | PM defined the scope in those terms. Recorded as an override to the what-not-how rule | 28 Jul 2026 |
| 15 | What counts as a closure | Two — the P75 deactivation closure that already runs, and a new exhaustion closure | A single new closure concept covering all stopping paths | "P75 (When CLOS moves the connection to Deactivated) - Already Live" plus a new signal for exhaustion. Documenting the live one keeps the closure picture complete in one place | 28 Jul 2026 |
| 16 | When the exhaustion closure may fire | Only when attempts are used up **and** no install task is active | Fire as soon as attempts run out | "all attempts to find the CSP is exhausted and none of the tasks are active." The second condition prevents a "nobody is coming" signal while a CSP is mid-install | 28 Jul 2026 |
| 17 | Ordering when cancellation and closure coincide | Cancellation first, then closure | Closure first; or no stated order | "P1 Ordering - Okay" — accepted the proposed order | 28 Jul 2026 |
| 18 | Acceptance-criteria example data | Keep the illustrative booking ids and CSP names | Replace with real production examples before sign-off | "AC Example Data - Let it be" | 28 Jul 2026 |
| 19 | Naming internal parameters in the glossary | Keep the mechanism-mapping row (P41, P74, P75, P78, P50) | Remove it for a fully mechanism-free PRD | "§8 mechanism-mapping row - Keep it" — lets engineering map product language to code without guessing | 28 Jul 2026 |
| 20 | The reason a CSP types when declining or reporting failure | Carry it through unaltered on those two triggers; send none on the two timeouts | Omit it; or normalise it into a fixed code set | "We also need the reason the CSP entered (while Declining or marking install failure as well)" | 28 Jul 2026 |
| 21 | Describing what the customer backend does with a signal | Removed every such statement — the spec ends at the send | Keep purpose framing like "so it can close the booking and start a refund" | "What happens to those signals is completely the customer backend responsibility" | 28 Jul 2026 |

## Open after sign-off

Not blockers, and not lint failures — the PRD's precedence rules P1 and P2 are stated and tested. These are two scenarios no precedence rule currently covers, raised twice during drafting and not yet decided:

| Scenario | Why it matters |
|---|---|
| A customer cancels their booking while a task cancellation is already in flight | T1's check is evaluated at trigger time. If the customer's cancellation lands a moment later, a task cancellation signal may already have gone out for a booking they just killed — the echo R1 MUST NOT (b) exists to prevent |
| An installation completes at the same moment a closure falls due | Nothing says which wins. Getting it wrong sends a "nobody is coming" closure on a booking that was just installed |

## Carried out of scope

| Item | Where it went |
|---|---|
| Blank-overwrite defect — a failed technician lookup sends an empty name and mobile, wiping what the customer already had | Needs its own ticket. Not raised anywhere else |
| P74 security-fee guard stall — a fee-paid connection whose install stalls sits with no timer of any kind | Found during the retry trace; outside this PRD's scope |
