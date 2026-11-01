# Handoff: Nuad Ankara — MVP (client + therapist + admin)

## Overview
Nuad is a two-sided mobile app for booking massage therapists in Ankara. Clients find verified therapists and send a booking request. The therapist accepts, declines or proposes other times. After acceptance a chat opens. The session happens at a studio or the client's home, the client pays the therapist **in cash**, and then reviews them. One account can hold both a client profile and a therapist profile and switch between them. An internal admin console verifies certificates and handles reports.

Launch shape: **invite-only pilot, ~20 therapists, Ankara.** The UI is in Turkish by default, with English planned as a toggle.

## About the design files
The files in `design/` are **design references built in HTML**. They are prototypes that show the intended look and behaviour; they are not production code to copy. Rebuild them in the target stack.

Repo `muygur/thai-app-tr` currently has only a README, so no stack has been chosen yet. **Recommended: React Native + Expo (TypeScript)**, giving one codebase for iOS and Android, with Expo Router for navigation. Backend: anything with auth, Postgres, realtime and storage (Supabase fits well: phone OTP auth, row-level security, realtime chat, storage for certificate scans).

To open the prototype, open `design/Nuad Ankara App.dc.html` in a browser; it needs `support.js` and `ios-frame.jsx` from the same folder. It renders inside a 402×874 iPhone frame.
- Prototype login code: **123456**.
- **You → Yönetim paneli → Durumlar** forces the empty, offline and error states.

## Fidelity
**High-fidelity.** Colours, type, radii, copy and flows are final for the MVP. Rebuild them pixel-close. Two areas are **likely to change** after therapist interviews (see `design/Nuad Terapist Gorusme Notlari.dc.html`):
- therapist onboarding steps 2–3 (disciplines, service area / who you accept, pricing)
- the filter options for availability

Build those screens data-driven: disciplines, durations, radius options and price presets come from config or the DB, never hard-coded in the UI.

---

## Business rules (source of truth)
These rules are not obvious from the screens. Enforce them on the server, not just in the UI.

**Accounts and auth**
1. Sign-in is by phone number + 6-digit SMS code (+90, mobile numbers start with 5, 10 digits after +90). Apple and Google are alternatives, but **always also verify a phone number** afterwards, because therapists visit homes.
2. OTP: 45 s resend cooldown, resend via SMS or voice call. Max 5 wrong attempts, then a 15-minute lockout. A new code invalidates the old one.
3. Sign-up order: phone → code → consent → first name + district → Home.
4. Consent:
   - **Terms** and **KVKK notice** are required. Each one can only be accepted after its text has been opened ("Okudum, onaylıyorum" inside the sheet accepts it). Store the version and timestamp for each consent.
   - **Health-data explicit consent** is optional. If it isn't given, the "Terapiste not" field in the booking request is hidden.
   - **Marketing** is optional.
5. Other users only ever see your first name. Phone numbers are never shown to the other party; all contact goes through in-app chat.
6. One account can hold a client profile and a therapist profile. The header segmented control and the "Müşteri moduna geç" chip switch between them. Unread badges are counted per role.

**Therapist visibility**

7. A therapist is **hidden from search** until at least one certificate is approved by an admin, or an admin approves an "experience review" (therapist has no certificate).
8. Rejecting a certificate does not hide a therapist who already has another approved one.
9. Therapists set which clients can find them: everyone, women only, or men only. This is **enforced by the server**: clients outside the setting never see the therapist, and the therapist never receives their requests.
10. Profiles show only the credential name and approval date. Scans are visible to admins only.

**Availability**

11. Weekly hours per day (on/off, from, to), a "closed today" toggle that resets at midnight, closed dates, and three rules:
    - **lead time:** 1 h / 2 h / 4 h / 1 day
    - **buffer between sessions:** none / 15 / 30 / 60 min
    - **booking horizon:** 1 week / 2 weeks / 1 month
12. Bookable slots = inside weekly hours, minus closed days, minus accepted sessions (+ buffer), respecting lead time and horizon. Clients only see these slots.
13. Changing hours or closing a day **never cancels an accepted session**. Show a warning when a change conflicts with one.

**Booking lifecycle**

14. Statuses: `requested` → `accepted` | `declined` | `time_proposed` → `confirmed` → `in_progress` → `completed` → `reviewed`. `cancelled` is possible from any status before `in_progress`.
15. The therapist should reply within 2 hours (shown as copy; auto-expiry is not designed yet, so treat it as a TODO).
16. Decline: the reason is optional (Already booked / Too far / Outside my disciplines / Not comfortable / no reason).
17. Propose another time: the therapist picks a day, then 1–3 slots generated from their own availability (busy slots are struck through). The client confirms one.
18. Completion: the therapist taps "Seans bitti" in the request card or the chat. Otherwise it **auto-completes 3 hours after the scheduled end**. Completion unlocks the client's review.
19. Payment: cash to the therapist after the session. The app collects nothing and takes no commission. Never show card UI.
20. "Book again" opens the therapist's profile with the last duration prefilled.

**Chat**

21. Chat exists only after the therapist accepts. It stays open until 24 h after the session.
22. Quick replies **fill the composer**; they never send directly. Enter sends on hardware keyboards. Which quick replies to show is an open question; ship a static list per role behind a flag.
23. Either side can **close** the conversation. Neither side can send while it's closed, and the person who closed it can reopen it.
24. **Report** goes to the admin queue with the full thread. The "Also block" toggle defaults to on for therapists and off for clients.
    - **Block:** the conversation closes and the blocked party can't message or request again. The blocker can unblock.
    - **Report without block:** the chat stays open.
    - The block state takes precedence over the closed state in the UI.

**Reviews**

25. 1–5 stars, a line describing each rating, multi-select tags, optional note. The client's first name is shown with the review. Safety issues go through report, not reviews.

**Admin**

26. Certificates queue: scan + OCR-extracted fields side by side, Approve / Reject. The confirmation copy states the consequence (e.g. "this makes X visible on the map for the first time").
27. Reports queue: reporter, reason, quoted messages, then Suspend / Warn / Dismiss. Suspend closes all open bookings for that account.

---

## Screens
All screens are 402 pt wide (iPhone 15/16 Pro), with a 62 pt status-bar safe area at the top. Tab bars are fixed to the bottom, padded 12/18/26, with `#ebddc5` fill and a 1 px top border at 10% ink. Scroll content leaves **120 pt bottom padding** so the last card clears the tab bar.

### Auth (z-index above everything)
- **Welcome:**
  - hero image (washed) over the top 47%, rounded 0 0 46 46
  - "Nuad" wordmark
  - H1 "Güvenebileceğin eller" (34/1.08)
  - three full-width pill buttons: primary "Telefon numarasıyla devam et"; Apple (ink fill `#201e1d`, cream text, Apple logo); Google (secondary, 4-colour G logo)
- **Phone:**
  - back button; step "Adım 1 / 3"
  - "+90" pill and a 56 pt pill input
  - live grouping "5XX XXX XX XX"; strips a leading 0
  - inline error if the number doesn't start with 5
  - CTA "Kod gönder" disabled until valid
  - if arriving from Apple/Google, a sage banner explains why the phone is still needed
- **Code:**
  - six 60 pt boxes over one hidden `one-time-code` input
  - the active box has an accent border, errors turn every border accent
  - countdown "0:45", then "Kodu tekrar gönder" / "Sesli arama ile al" chips
  - error copy counts the remaining attempts
- **Consent:**
  - four rows: round 28 pt check, text, a "Zorunlu" / "İsteğe bağlı" label, and a "Metni oku" chip
  - tapping the check of an unread required text opens the text first
  - while incomplete, the CTA area shows a sand pill saying what's missing; when complete, the primary CTA "Kabul et ve devam et" appears with an accent shadow
- **Legal sheet:** bottom sheet (max 82% height) with titled summary cards, "Okudum, onaylıyorum" and "Onaylamadan kapat". The real legal texts must be supplied by a lawyer.
- **Name + district:** name input, district chips, then "Başla".

### Client (tabs: Ana sayfa, Harita, Randevular, Sen)
- **Home:** greeting, search, discipline chips, "Terapistlerin" (regulars), "Yakınındaki müsait terapistler" list cards (photo, name, rating, distance, next free slot, price / 60 min, "İstek gönder"). Empty state: "Bu filtrelere uyan kimse yok" + clear filters.
- **Map:** pins show photo + first name only. Tapping a pin raises a card with time and price.
- **Filters sheet:**
  - Disciplines: **multi-select**; "Tümü" clears the selection.
  - Availability: Her zaman / Bugün / Şimdi müsait (under review).
  - Price per 60 min: **two range sliders**, ₺500–₺1.500+, step 50, min 100 apart.
  - Where: any / studio / home visit.
- **Therapist profile:** photo, rating, years of experience, price, approved credentials list, disciplines, reviews, "Randevu iste".
- **Booking request:** treatment, duration (60/90/120), day, time (from the availability engine), "Terapiste not" (textarea, 400 chars, only this therapist sees it), "Nakit, seans sonunda" line, "İsteği gönder". Then the "İstek gönderildi" confirmation.
- **Bookings list / detail:** grouped by status; actions per status (Mesaj, Değerlendir, İptal, Tekrar randevu al). Cancel sheet with optional reason.
- **Review:** see rule 25.
- **You:** name, district, saved therapists, payment ("Nakit, seans sonunda"), language, admin entry (internal), "Terapist ol", "Çıkış yap".

### Therapist (tabs: İstekler, Sertifikalar, Profil)
- **Onboarding 1–3:**
  1. name, photo, district, one-line bio
  2. disciplines (multi-select), price per 60 min (presets ₺750/₺900/₺1.050/₺1.200 **or free entry** "Kendin yaz"), years of experience
  3. service area (studio / home / both), travel radius, clients accepted

  Then the certificates screen.
- **Requests inbox:**
  - header chip "Müşteri moduna geç" with an unread badge
  - request cards with client first name, time, duration, place and note
  - actions: Kabul et / Başka saat / Reddet
  - accepted cards: Mesaj, plus "Seans bitti" on the day of the session
  - empty state that explains your current visibility
- **Propose time sheet:** 5-day strip (closed days dimmed), 4-column slot grid, up to 3 picks, CTA "N saat öner".
- **Decline sheet:** see rule 16.
- **Certificates:** list with status (Onaylandı / İncelemede / Reddedildi + reason), "Sertifika ekle", "Sertifikan yok mu? Deneyimine göre inceleme iste".
- **Upload:** photo or file (PDF/image, max 10 MB), then review the OCR fields (Belge, Veren okul, Veriliş, Sertifika numarası). Each field is editable. Then "İncelemeye gönder".
- **Profile:** visibility banner ("Aramada görünüyorsun" / "Bir sertifika onaylanana kadar gizlisin"), price, disciplines, clients accepted, **"Çalışma saatleri" row with a summary** that opens the availability editor.
- **Availability editor:** see rule 11. Weekly rows are a toggle plus a time pill that opens the hours sheet (from/to chips + "Bu saatleri tüm açık günlere uygula"). A "Bir gün kapat" sheet with a 10-day grid. Conflict warnings are shown in an `#ffe1d0` box with `#643312` text.

### Shared
- **Chat thread:**
  - booking summary strip (sage) at the top, then bubbles: mine accent, theirs sand, system centred
  - quick-reply chips above the composer, then the composer with attach and send
  - closed / blocked footer card with "Sohbeti yeniden aç" / "Engeli kaldır"
- **Chat menu:**
  - Randevuyu gör
  - Seansı bitmiş olarak işaretle (therapist only)
  - Sohbeti kapat (hidden while closed or blocked)
  - Bildir…
- **Report sheet:** reason chips (Taciz / Cinsel talep / Hakaret / Başka bir şey), an "Ayrıca engelle" toggle with consequence copy, CTA "Bildir" / "Bildir ve engelle".
- **Attach sheet:** photo, location, re-send booking details.
- **Offline banner:** sits above the tab bar on every screen, with "Tekrar dene". A failed send shows on the message with retry.

### Admin (internal)
Tabs: Certificates, Accounts (reports), States (prototype only, do not build).

---

## Design tokens (Organic design system)
Colours:
- bg `#f5ead8` · surface `#ebddc5` · text `#201e1d` · accent `#c67139` · accent-2 (sage) `#7a8a5e`
- accent ramp 100–900: `#fff2eb #ffe1d0 #ffc6a5 #f6a06b #d67f48 #b2622d #8c491a #643312 #402310`
- sage ramp 100–900: `#f0fae1 #e1eecc #ccdbb2 #aebf92 #8fa073 #728157 #56633f #3d472b #272e1b`
- neutral ramp 100–900: `#f9f4ed #eee7db #dcd3c4 #c0b6a5 #a19786 #82796a #645c50 #474238 #2e2b25`
- secondary text is ink at 60–72% opacity; the divider is ink at 10–16%

Colour usage:
- **Accent `#c67139` is for fills and icons only.** Body-size accent text uses `#8c491a` (accent-700).
- Success / info boxes: `#e1eecc` fill with `#3d472b` text.
- Warning boxes: `#ffe1d0` fill with `#643312` text.

Type:
- Headings: **Corben 700** (Google Fonts).
  - The design system specifies Caprasimo, but Caprasimo lacks the Turkish glyphs ı İ ğ Ğ ş Ş. Do not use it.
  - Set `lang="tr"` (or the locale) so uppercase transforms produce İ.
  - Sizes: H1 34, H2 28–30, H3 20–22, H5 14–15.
- Body: **Figtree** 400/600/700, sizes 11–17, line-height 1.45–1.55.

Radii: inputs and buttons 999 (pills) · cards 24–30 · sheets 34 34 0 0 · chips 999.
Spacing: screen gutter 22 · card padding 14–20 · list gap 8–16 · DS scale 4.4 / 8.8 / 13.2 / 17.6 / 26.4 / 35.2.
Shadows: sm `0 1px 2px rgba(46,43,37,.14)` · md `0 3px 10px rgba(46,43,37,.16)` · lg `0 12px 32px rgba(46,43,37,.22)`.
Toggles: 48×28 track (on `#c67139`, off ink 22%), 22 pt cream knob.
Icons: Lucide, stroke-width 2.75.
Touch targets ≥ 44 pt.

## Data model (suggested)
- `users` (id, phone, first_name, last_name?, district, locale, created_at)
- `consents` (user_id, kind, version, accepted_at)
- `therapist_profiles` (user_id, bio, price_60, years, service_mode, radius_km, accepts, visible)
- `therapist_disciplines` (therapist_id, discipline_id)
- `disciplines` (config)
- `availability_weekly` (therapist_id, weekday, on, from, to)
- `time_off` (therapist_id, date)
- `booking_rules` (therapist_id, lead_min, buffer_min, horizon_days)
- `paused_until`
- `certificates` (id, therapist_id, file_url, ocr_json, status, reject_reason, reviewed_by, reviewed_at)
- `bookings` (id, client_id, therapist_id, treatment, duration_min, start_at, place, note, status, proposed_slots[], completed_by, timestamps)
- `threads` (booking_id, closed_by?, closed_at?)
- `messages` (thread_id, sender_id | system, body, attachment, sent_at, failed)
- `blocks` (blocker_id, blocked_id)
- `reports` (id, reporter_id, reported_id, thread_id, reason, status, action, admin_id)
- `reviews` (booking_id, stars, tags[], note)

## Not designed yet (backlog)
- push notifications and quiet hours
- the cancellation policy and late-cancel flagging
- auto-expiry of unanswered requests
- therapist earnings/history
- safety (share a session, emergency contact)
- certificate expiry and re-checks
- the English toggle (all copy should go through i18n from day one)
- the welcome animation after the first approval
- how quick replies are chosen

## Files
- `design/Nuad Ankara App.dc.html` is the full interactive prototype; all screens and states are in here.
- `design/Nuad MVP Status.dc.html` is the inventory of screens and gaps.
- `design/Nuad Terapist Gorusme Notlari.dc.html` holds the open questions for therapist interviews. Screens touching these may change.
- `design/support.js`, `design/ios-frame.jsx` and `design/_ds/` are needed to open the prototype in a browser.
