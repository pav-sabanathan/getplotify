# Decision: Reset Flag Approach for Developer Testing Utility

**Version:** Pre-V2 Polish  
**Date:** March 2026  
**Status:** Resolved

---

## The Question

How should the "Reset app data" developer utility 
handle the app's sample data seeding logic to ensure 
a genuine clean state is achieved on reset?

---

## The Problem

A "Reset app data" button was added as a testing 
utility to allow the empty state and onboarding flow 
to be tested without manually clearing browser storage. 

On first use, the reset appeared to work — localStorage 
was cleared and the page reloaded. However the sample 
shows immediately reappeared after the reload, making 
the reset functionally useless for testing purposes.

**Root cause identified by Lovable:**

*"The page reloaded but shows are back — this is 
because loadShows() returns SAMPLE_SHOWS when there's 
no localStorage key (first visit). The reset clears 
localStorage, so the reload sees no key and re-seeds 
with samples. I need to set a flag during reset so 
the app knows it was intentionally cleared."*

In plain English: the app uses the absence of a 
localStorage key as a signal that this is a first 
visit, and automatically seeds sample shows in that 
case. Clearing localStorage recreated exactly those 
conditions, causing the app to re-seed itself 
immediately after every reset.

---

## Options Considered

**Option 1 — Remove sample data entirely**
Remove the pre-seeded sample shows so there is 
nothing to re-seed on first visit. 

Rejected — the sample shows are valuable for 
demonstrating the product to new visitors and 
should remain for the public-facing product.

**Option 2 — Clear storage without reloading**
Clear localStorage without triggering a page reload, 
relying on React state updates to reflect the change.

Rejected — without a reload the onboarding flow 
would not re-trigger correctly as it checks for 
first visit on initial mount.

**Option 3 — Write a reset flag to localStorage**
During a reset, write a specific flag key to 
localStorage (e.g. `plotify_was_reset: true`) before 
reloading. On reload, the app checks for this flag 
first — if present, it skips sample data seeding, 
deletes the flag, and treats the session as 
intentionally empty.

Selected — this cleanly separates intentional 
resets from genuine first visits without removing 
the sample data or breaking the reload behaviour.

---

## The Decision

**Option 3 — Write a reset flag to localStorage**

A dedicated reset flag written to localStorage 
immediately before the page reload tells the app 
to skip sample data seeding on the subsequent load. 
The flag is consumed and deleted on that reload so 
it only affects the immediate post-reset visit.

---

## The Outcome

The reset button now correctly produces a genuine 
empty state with no shows, allowing the onboarding 
flow and empty states to be tested accurately without 
requiring manual browser storage clearing or an 
incognito window.

---

## Broader Learning

This issue is a good example of emergent complexity 
in stateful applications — a feature that seems 
simple (clear the data) interacts with existing 
initialisation logic in an unexpected way. The fix 
required understanding not just what to clear but 
how the app decides what to load on startup.

For a PM this is a valuable pattern to recognise: 
seemingly simple reset or undo features often have 
hidden complexity when the app has initialisation 
logic that runs on load. Worth flagging early in 
any future feature that involves clearing or 
resetting user state.
