# What to build next

_Started 2026-09-11 13:30 UTC_

---

## User

<artifact-content-authored-by-others/>
Continuing from "Mobile app design".

## Assistant

We've been building a dual-sided massage-booking app (client + therapist) in Ankara using the Organic design system. The client side (Home, Map with availability pins, Filters, Profile, Booking flow, Bookings list) is complete and working. We just finished Phase 1 of the therapist side: onboarding (profile + disciplines + service area + client gender preference), certificate upload with OCR-style field extraction, verification status screen, and request inbox with decline reasons.

Key decisions made:
- Chat opens only after therapist accepts a request, with therapist-side ban/close option for harassment.
- Therapists stay hidden until at least one certificate is approved, but can request admin review on experience alone.
- Certificate approval is both per-certificate and per-therapist (any valid cert unlocks visibility).
- Payment deferred entirely for now — no payment logic in this loop.
- Decline requires no reason but reasons are offered as a courtesy.

Current state: Therapist onboarding, certificate upload, verification status, and request inbox are in the prototype. The therapist Profile tab now displays onboarding choices. Status bar fixed to stay above overlays; inbox heading pluralizes.

Next: Phase 2 (chat thread after acceptance, with pinned booking card, quick replies, attachments, and therapist close/ban option). Then Phase 3 (client-side status changes and post-session review), Phase 4 (admin certificate queue and customer account review), and empty/error states across all flows.

Files involved: **Nuad Ankara App.dc.html** (main prototype, all three directions also in **Nuad Home Directions.dc.html** for reference).

## User

what do we build next?

## Assistant

Next up is **Phase 2: the chat thread**, which unlocks right after a therapist accepts. Concretely:

- Therapist accepts from inbox → thread appears on both sides, with the booking pinned at top (service, date/time, address type, price placeholder).
- Quick replies for the common exchanges (confirm address, running late, what to prepare).
- Attachment affordance (photo/location), no real upload.
- Therapist overflow menu: close conversation, report/ban for harassment.
- Client side: same thread, no ban option, but a report link.

After that: Phase 3 (client status changes + post-session review), Phase 4 (admin cert queue + customer account review), then empty/error states.

Want me to start on chat, or reorder — some people prefer getting client-side status/review done first since it closes the booking loop?

## User

start on chat, yeah then we can closes the booking loop.

## Assistant

_[tool: grep]_

_[tool: read_file]_

_[tool: read_file]_

_[tool: read_file]_

_[tool: read_file]_

_[tool: read_file]_

Now building the chat thread.

_[tool: dc_html_str_replace]_

_[tool: dc_html_str_replace]_

Now the chat overlay itself.

_[tool: dc_html_str_replace]_

_[tool: dc_js_str_replace]_

_[tool: read_file]_

_[tool: read_file]_

_[tool: grep]_

_[tool: dc_js_str_replace]_

_[tool: grep]_

_[tool: dc_js_str_replace]_

Now wiring the entry points.

_[tool: dc_js_str_replace]_

_[tool: dc_js_str_replace]_

_[tool: show_html]_

_[tool: ready_for_verification]_

Chat is in. Therapist: accepting a request now seeds a thread and the card gets a "Message …" row with the last line; inside, a pinned booking bar, quick replies, attachments (photo / location / re-send booking), and an overflow with close conversation and report-and-block. Client: confirmed bookings get a Message chip, same thread, report only — no block.

Next, Phase 3 to close the booking loop: client-side status changes (therapist on the way, completed, cancel) and the post-session review.

## User

Found issues — fixing…

## Assistant

_[tool: dc_js_str_replace]_

_[tool: dc_html_str_replace]_

_[tool: ready_for_verification]_

Fixed: the menu and attachment backdrops now only dismiss on a click on the backdrop itself, so report/block and close-conversation reach their sheets and the closed-thread state is exercisable.

## User

Found issues — fixing…

## Assistant

_[tool: dc_js_str_replace]_

_[tool: ready_for_verification]_

Client closed-thread banner now reads "Reported to an admin". (The two overlap findings are the pre-existing Home hero placeholder labels, unrelated to chat.)

## User

go with the phase 3
addition: on the map. face of the people(photo) and the name is enough. or maybe photo is enough rather than hours or price

## Assistant

_[tool: grep]_

_[tool: read_file]_

_[tool: grep]_

_[tool: read_file]_

_[tool: dc_html_str_replace]_

_[tool: dc_js_str_replace]_

_[tool: dc_js_str_replace]_

Now the bookings card, detail, cancel and review screens.

_[tool: dc_html_str_replace]_

_[tool: dc_html_str_replace]_

_[tool: dc_html_str_replace]_

_[tool: dc_js_str_replace]_

_[tool: dc_js_str_replace]_

_[tool: dc_js_str_replace]_

_[tool: ready_for_verification]_

Map pins are now the therapist's photo with their first name under it — no clock, no price; the time and price stay on the selected card at the bottom.

Phase 3 in: bookings list seeds all five states (awaiting, confirmed, on the way today, completed, reviewed), cards are tappable into a booking detail with a status timeline, and the detail carries the right action per state — message, cancel (with optional reason sheet), leave a review, or book again. Review screen has stars with a per-rating line, what-stood-out tags, optional note, and posts back to the card as "You rated ★★★★★".

Next: Phase 4, admin certificate queue and customer account review — then empty and error states.

## User

nice, question, 1- what makes the session compete?   2- switching from client to therapist opens a new therapist page where i enter my information. what if I already a therapist profile too?(sinci the current app is stateless, i think it just cleans the memory, but afteri become a therapist, i am ready to switch easiliy) moreover .  maybe there should be easier way to switch between client-therapists? 3- when i click Book XX again, it just returns me to Home. (maybe this is problem of the logic, not the UI design) 

so question last 5- what else do we need before we go into claude code

## Assistant

_[tool: grep]_

Answers:

**1 — What completes a session.** Right now nothing does; I seeded "Completed" as data. Options, cheapest to strictest: (a) auto-complete at end time + duration, (b) therapist taps "Session finished" in the thread, (c) both sides confirm. My recommendation is (b) with (a) as a fallback after a few hours, since the therapist is the one who knows it actually happened. The review prompt fires off that event.

**2 — Switching.** The logic already keeps your therapist state: onboarding only runs while `obStep` is unfinished, so after you complete it, switching lands straight on the inbox. What's missing is the affordance — it's one buried row in Profile, and there's no signal you have both roles. Fix: a role chip in the header (photo + "Client / Therapist") that swaps instantly once both roles exist, and keeps the buried row as the "become a therapist" entry only when you have no therapist profile. Unread request/message badges should show on the role you're not in.

**3 — Book again.** Correct, that's logic, not design: it drops you at Home instead of opening that therapist's profile with the booking sheet prefilled. Small fix.

**5 — Before Claude Code.** Still open: Phase 4 (admin cert queue, customer account review), empty and error states everywhere, the session-completion rule above, notifications/badging, cancellation rules near the session time (who eats what, once payment exists), and the therapist's own availability editing — right now availability is display-only. Also worth writing down: what happens when a certificate is rejected after the therapist is already live.

Want me to do 1b + 2 + 3 now as one pass, then Phase 4?

## User

1-  b with a as a fallback. . yes do one pass. then phase 4

## Assistant

_[tool: grep]_

_[tool: read_file]_

_[tool: grep]_

_[tool: read_file]_

_[tool: grep]_

_[tool: read_file]_

_[tool: read_file]_

_[tool: dc_html_str_replace]_

_[tool: dc_html_str_replace]_

_[tool: dc_html_str_replace]_

_[tool: dc_html_str_replace]_

_[tool: dc_html_str_replace]_

_[tool: dc_html_str_replace]_

Now the logic.

_[tool: dc_js_str_replace]_

_[tool: dc_js_str_replace]_

_[tool: dc_js_str_replace]_

_[tool: dc_js_str_replace]_

_[tool: dc_js_str_replace]_

_[tool: dc_js_str_replace]_

_[tool: dc_js_str_replace]_

_[tool: ready_for_verification]_

All three in:

**Completion** — today's accepted request (Aylin Şen, 17:30) has "Session finished" on the card and in the chat menu, with the fallback stated underneath: marked finished automatically 3 hours after the end time. Tapping it flips the client's booking to Completed, so "Leave a review" appears on their side and a system line lands in the thread.

**Switching** — both profiles now carry a Client / Therapist segmented control with a count badge on the side you're not in (new requests for therapist, sessions awaiting review for client). "Become a therapist" only shows when there's no therapist profile yet; once onboarding is done, switching is one tap and keeps your state.

**Book again** — now opens that therapist's profile with the duration carried over instead of dumping you at Home.

Phase 4 next: admin certificate queue and customer account review.

## User

i dont see session finished button

## Assistant

_[tool: read_file]_

_[tool: grep]_

Data's there — the live instance was still holding pre-edit state from a hot reload. Reopening remounts it.

_[tool: ready_for_verification]_

Reloaded. The today's-session card is Aylin Şen · Today 17:30 — third in the Requests list, above Deniz. Deniz is Mon 15 Sep, so it correctly has no finish button, only the message row.

## User

are we ready for phase 4? if we ready, consider the usage of token for this session, then go!

## Assistant

_[tool: dc_html_str_replace]_

_[tool: dc_html_str_replace]_

_[tool: dc_js_str_replace]_

_[tool: dc_js_str_replace]_

_[tool: dc_js_str_replace]_

_[tool: ready_for_verification]_

Phase 4 is in — reachable from client Profile → Admin console.

**Certificates tab**: each item shows the scan, the extracted fields, and a flag when something is unreadable (Burak's is abbreviated and undated). Approve/Reject states the consequence before you act — first approval makes the therapist visible, rejecting one of several doesn't hide a profile that's already live.

**Accounts tab**: reports arriving from chat, each with the reason, who reported, and the two thread lines that matter. Suspend / Warn / Dismiss, each with its settled state.

Both tabs have empty states and count pips. Remaining before Claude Code: empty and error states across the client and therapist flows, notification badging beyond the role switch, therapist availability editing, and the cancellation-window rule.

## User

go on. questiosn: what is notification badging and how to test empty and error states?

## Assistant

Notification badging = the unread counts. Right now only the role switch has one; the missing pieces are per-tab dots (Bookings when a therapist replies or a session needs review, Requests when a new one lands) and the OS-level push that brings you back in. I'll add the in-app ones.

For testing states, I'll add a States tab in the admin console that empties or breaks each flow on demand.

_[tool: read_file]_

_[tool: read_file]_

_[tool: read_file]_

_[tool: dc_html_str_replace]_

_[tool: dc_html_str_replace]_

_[tool: dc_html_str_replace]_

_[tool: dc_html_str_replace]_

Now the States tab, tab badges, and the logic.

_[tool: dc_html_str_replace]_

_[tool: dc_html_str_replace]_

_[tool: dc_html_str_replace]_

_[tool: dc_js_str_replace]_

_[tool: dc_js_str_replace]_

_[tool: dc_js_str_replace]_

_[tool: dc_js_str_replace]_

_[tool: dc_js_str_replace]_

_[tool: grep]_

_[tool: read_file]_

_[tool: read_file]_

_[tool: dc_js_str_replace]_

_[tool: dc_js_str_replace]_

_[tool: dc_js_str_replace]_

_[tool: dc_js_str_replace]_

_[tool: dc_js_str_replace]_

_[tool: dc_js_str_replace]_

_[tool: dc_js_str_replace]_

_[tool: ready_for_verification]_

Answers plus the build:

**Badging** is the unread count. Bookings now carries a count for sessions waiting on a review; the role switcher already badges new requests on the therapist side and reviews on the client side. Push notifications live outside the app, so they're a spec line rather than a screen.

**Testing states** — Profile → Admin console → States. Five toggles: no therapists match, no bookings, no requests, connection lost, and requests fail to send. Flip one, leave the console, and the flow renders that state. They're real state, not mock screens, so the empty list still clears with its own Clear filters button and the offline banner clears with Retry.

Empty states added to client search results, bookings, and the therapist inbox; the offline banner sits above every screen with retry, and a failed send surfaces it rather than silently dropping the message.

## User

Found issues — fixing…

## Assistant

_[tool: dc_js_str_replace]_

_[tool: dc_js_str_replace]_

_[tool: dc_html_str_replace]_

_[tool: dc_html_str_replace]_

_[tool: ready_for_verification]_

Three fixes: Accounts no longer highlights while States is open, the offline banner moved to sit above the tab bar so it stops covering headers and the search field, and the empty-bookings button no longer wraps.

