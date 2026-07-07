# invites.blog — Product & Engineering Specification

**Version:** 2.0
**Prepared for:** invites.blog
**Primary stack:** Angular 22 + ASP.NET Core / .NET 10
**Database:** PostgreSQL 17+
**Primary architecture:** Modular monolith first, service-ready later
**Primary domains:** `invites.blog`, `me.invites.blog`, `api.invites.blog`, `assets.invites.blog`

**Guiding principles:**
1. Maximum user-friendliness — links should "just work."
2. Never force registration. No accounts, no passwords. Access is possession-based (tokens + magic links).
3. Animation is the product, but content always works without it.

---

## 1. Executive Summary

`invites.blog` is a premium animated invitation platform where inviters create beautiful, role-aware, personalized digital invitations and send them through email, WhatsApp, Viber, Telegram, or direct phone-linked channels.

The platform has two major experiences:

1. **Public inviter experience at `invites.blog`**
   - Magazine-style landing page.
   - Animated template showcase.
   - Template selection and customization.
   - Comprehensive template designer for custom templates.
   - Guest list upload through Excel.
   - Guest validation and role/gender personalization.
   - Delivery channel selection.
   - Payment.
   - Invite dispatch.
   - Post-payment campaign dashboard (delivery report, RSVPs, add guests, resend).

2. **Private invitee experience at `me.invites.blog`**
   - Zero-login invite viewing via secure token links.
   - Optional OTP login (phone or email) for the invite inbox.
   - Animated, scroll-driven invite viewing experience.
   - Role-based and gender-based invite content.
   - RSVP and guest actions.

The differentiator:

> **Animated, personalized, role-aware invitation experiences with a private invite inbox — and no forced sign-up for anyone.**

---

## 2. Product Vision

### 2.1 Mission

To make sending and receiving digital invitations feel personal, premium, secure, and memorable.

### 2.2 Positioning

**invites.blog** is a digital invitation magazine and builder for people who want more than a flat image, PDF card, or dull RSVP form.

### 2.3 Tagline Options

- Invitations with a story.
- Beautiful invites. Delivered personally.
- Animated invitation pages for every guest.
- Your event, told beautifully.
- Where invitations become experiences.

### 2.4 Core Promise

An inviter can create a stunning invite, upload guests, personalize content by role or gender, pay, and send unique invitation links through modern communication channels — without ever creating an account.

An invitee can open their link and see their personalized animated invite instantly, RSVP with zero login, and optionally verify once (phone or email OTP) to keep all received invites in a beautiful inbox.

---

## 3. Domain Model Overview

### 3.1 Domains

| Domain | Purpose |
|---|---|
| `invites.blog` | Public landing, template magazine, builder, designer, payment flow, campaign dashboard |
| `me.invites.blog` | Invite viewing, optional OTP login, invite inbox |
| `api.invites.blog` | Backend API |
| `assets.invites.blog` | CDN/static asset domain; serves template packages under strict CSP |

### 3.2 Main Actors

| Actor | Description |
|---|---|
| Visitor | Anyone browsing templates on `invites.blog` |
| Inviter | Person creating and sending invitations (no account required) |
| Invitee | Recipient of an invitation |
| Designer | An inviter who builds a custom template and may submit it to the gallery |
| Admin | Platform operator managing templates, submissions, channels, payments, abuse, and analytics |
| Delivery Provider | Email, WhatsApp, Viber, Telegram, SMS, or similar channel |
| Payment Provider | Stripe, BML payment gateway, Bank of Maldives, Ooredoo Pay, or equivalent |

---

## 4. Product Experiences

## 4.1 Public Landing Page: `invites.blog`

### 4.1.1 Purpose

The root domain should feel like a magazine of invitation templates, not a dry SaaS homepage. The landing page showcases templates like featured editorial cards, animated previews, and categories.

### 4.1.2 Page Sections

1. **Hero Section**
   - Large animated headline.
   - Rotating or sliding template previews.
   - CTA: `Create an Invite`
   - Secondary CTA: `View Templates`

2. **Template Magazine Section**
   - Large editorial grid; each card animates on hover/scroll.
   - Categories: Wedding, Engagement, Ceremony, Business Meeting, Corporate Event, Graduation, Birthday, Private Dinner, Religious Gathering, Launch Event, Class/Workshop, Custom Event.
   - Community-designed templates carry a "Designed by {name}" badge.

3. **Animated Template Slider**
   - Horizontal carousel of miniature invitation animations.
   - Autoplay, pause on hover, respects reduced-motion settings.

4. **How It Works**
   - Select or design a template → Customize → Upload guest list → Pay → Send.

5. **Personalization Section**
   - Demonstrates role/gender dynamic content (bridesmaid floral variant, groomsman formal variant, VIP schedule block).

6. **Pricing Preview**
   - `$5 minimum` including 50 invites, then `$1 per 10 invites`.
   - "Design & share your own template → 50% off" teaser.

7. **Delivery Channels**
   - Email, WhatsApp, Viber, Telegram, direct link, phone-linked delivery.

8. **Footer**
   - Guide, Pricing, Privacy, Terms, Contact, Status.

### 4.1.3 Design Direction

- Premium magazine layout, smooth scroll animations, large image-driven templates, editorial typography, subtle paper/envelope metaphors.
- Use animation carefully; restraint over spectacle.

---

## 4.2 Templates

### 4.2.1 Template Card Fields

| Field | Description |
|---|---|
| Name | Human-readable template name |
| Slug | URL-safe identifier |
| Category | Wedding, meeting, ceremony, etc. |
| Preview Image | Static thumbnail (auto-generated headless render) |
| Preview Animation | Short animation preview (auto-generated) |
| Supported Variables | Dynamic placeholders declared in the manifest |
| Supported Roles | Optional role presets |
| Supported Gender Variants | Optional gender-based content blocks |
| Premium Flag | Free or paid template |
| Designer Attribution | Platform or community designer name |
| Theme Config | Colors, fonts, layout parameters |
| Version | Semver; campaigns pin the version they were created with |

### 4.2.2 Template Variables

```text
{{guest.name}}
{{guest.email}}
{{guest.phone}}
{{guest.role}}
{{guest.gender}}
{{event.title}}
{{event.date}}
{{event.time}}
{{event.venue.name}}
{{event.venue.address}}
{{inviter.name}}
{{inviter.phone}}
{{inviter.email}}
{{rsvp.link}}
{{invite.link}}
```

Custom variables may be defined per template and map to extra Excel columns.

### 4.2.3 Conditional Rendering

Templates support conditional content blocks driven by guest metadata:

```text
IF guest.role == "bridesmaid"  → show bridesmaid instructions
IF guest.gender == "male"      → show male dress code
ELSE IF guest.gender == "female" → show female dress code
```

Conditions are stored as JSON rules (§12), never executable scripts. Every conditional block must define default/neutral content so guests with `unspecified` values always see a complete invite.

---

## 4.3 Template Editor

### 4.3.1 Editor Goals

Let inviters customize a chosen template without design skills. (The full custom template *designer* is specified in §6.)

### 4.3.2 Editable Areas

| Area | Examples |
|---|---|
| Event title | "Aisha & Adam Wedding" |
| Subtitle | "Together with their families…" |
| Date/time | Event date, start time, end time |
| Venue | Name, address, map link |
| Description | Main invite body |
| Dress code | Formal, traditional, casual, etc. |
| Schedule | Ceremony, dinner, reception |
| RSVP Settings | Deadline, allowed statuses |
| Media | Cover image, couple image, logo |
| Theme | Colors, font pair, animation intensity |
| Role Blocks | Bridesmaid/groomsman/VIP/family notes |
| Gender Blocks | Male/female/neutral dress instructions |
| Delivery Message | Text used in email/chat delivery |

### 4.3.3 Editor Layout

- Left panel: editable fields.
- Center: live preview (the real template iframe, §5.5).
- Right panel: template variants and dynamic rules.
- Top bar: Save draft, Preview as guest, Next.

### 4.3.4 Preview Modes

Preview as: generic guest, specific uploaded guest, bridesmaid, groomsman, male guest, female guest, VIP guest, no role.

---

## 4.4 Guest List Upload

### 4.4.1 Upload Method

Excel upload. Supported: `.xlsx` (preferred), `.xls` optional, `.csv` future.

### 4.4.2 Excel Guide Page

Public page at `https://invites.blog/guide` explaining required/optional columns, an example download file, common mistakes, phone/email formatting, and role/gender examples. (Copy in Appendix §26.)

### 4.4.3 Required Excel Structure

First row = headers.

| Column | Header | Required | Description |
|---|---|---:|---|
| A | email | Conditional | Required if phone is missing |
| B | phone | Conditional | Required if email is missing |
| C | name | Optional | Guest display name |
| D | role | Optional | bridesmaid, groomsman, vip, family, speaker, etc. |
| E | gender | Optional | male, female, neutral, unspecified |

At least one of `email` or `phone` is required per row.

### 4.4.4 Phone Number Normalization

Phone identity is load-bearing (inbox matching, delivery, dedupe):

1. **Canonical format: E.164** (`+9607777777`), stored in `phone_e164` columns everywhere (guests, inviters, OTP challenges). `phone_raw` retained on guests for error reports.
2. Normalize with **libphonenumber** (`libphonenumber-js` in Angular, `libphonenumber-csharp` in the API) at three points: Excel parse, inviter details form, OTP login form.
3. The upload flow asks for a **default country** (pre-selected from campaign locale, e.g. MV) so local-format numbers (`7777777`) normalize correctly; numbers starting with `+` keep their own country code.
4. The OTP login input is a country-code picker + national number field.
5. Inbox matching, duplicate detection, and blocklists compare only `phone_e164`.
6. libphonenumber "impossible" numbers → row error; "valid but unusual" → warning, keep.

### 4.4.5 Validation Rules

| Rule | Behavior |
|---|---|
| Missing email and phone | Reject row |
| Invalid email | Warning or reject based on delivery channel |
| Invalid/impossible phone | Warning or reject based on delivery channel |
| Duplicate email/phone (E.164 compared) | Merge or flag duplicate |
| Missing name | Fallback: "Guest" |
| Unknown role | Accept as custom role, warn |
| Unknown gender | Store as `unspecified` |
| Empty rows | Ignore |
| Merged cells | Reject file with clear message |
| Scientific-notation phone numbers | Attempt recovery, warn user |

### 4.4.6 Upload Results Screen

Show: total rows, valid guests, invalid rows, duplicates, missing phone count, missing email count, role distribution, gender distribution, downloadable error report. Continue requires ≥1 valid guest.

---

## 4.5 Venue Type Selection

### 4.5.1 Venue/Event Types

Wedding, Engagement, Ceremony, Meeting, Corporate Event, Religious Event, Graduation, Birthday, Private Dinner, Workshop, Conference, Launch Event, Other.

### 4.5.2 Venue Fields

| Field | Required |
|---|---:|
| Venue type | Yes |
| Venue name | Yes |
| Address | Optional |
| Google Maps link | Optional |
| City/island/country | Optional |
| Room/hall name | Optional |
| Arrival instructions | Optional |
| Parking instructions | Optional |
| Dress code | Optional |

---

## 4.6 Inviter Details & No-Registration Access

### 4.6.1 Fields (collected before payment)

| Field | Required | Purpose |
|---|---:|---|
| Name | Yes | Displayed to invitees |
| Phone | Yes | Contact and delivery identity (stored E.164) |
| Email | Yes | Receipt, magic links, support |
| Organization | Optional | Corporate events |
| Billing name | Optional | Receipt |
| Billing country | Optional | Tax/payment support |

### 4.6.2 Access Model: token + magic link, never an account

1. Clicking "Create an Invite" creates a campaign instantly — **no sign-up screen**.
2. The API returns a `campaignAccessToken` (256-bit random; only its hash is stored). The Angular app keeps it in memory + `localStorage` and sends it as a Bearer token on every campaign endpoint.
3. **All** campaign write endpoints require this token and verify it maps to that campaign. Campaign IDs alone grant nothing.
4. When the inviter enters their email, send a **"Resume your invite" magic link** containing the access token — the recovery path if the browser is cleared. Zero extra user steps, since email is required before payment anyway.
5. After payment, email the **campaign dashboard magic link** (delivery report, RSVPs, add guests, resend). Long-lived, campaign-scoped, revocable, regenerable via a public "resend my dashboard link" form that emails (never displays) the link.
6. Future (Should-Have): entering the same email + email OTP shows all past campaigns — multi-campaign continuity, still no password.

---

## 4.7 Payment Flow

### 4.7.1 Pricing Model

```text
Minimum charge: $5
Includes: 50 invites
Extra invites: $1 per 10 invites
Community designer rate (§6.5): extra invites at $1 per 20 invites (50% off)
```

The minimum charge exists because payment providers charge transaction fees; without it, small orders lose money.

### 4.7.2 Pricing Formula

```text
includedInvites = 50
minimumPrice   = 5
blockSize      = hasDesignerDiscount ? 20 : 10
extraInvites   = max(0, inviteCount - includedInvites)
extraBlocks    = ceil(extraInvites / blockSize)
totalPrice     = minimumPrice + extraBlocks
```

### 4.7.3 Payment States

| State | Description |
|---|---|
| Draft | Campaign created but unpaid |
| PendingPayment | Checkout started |
| Paid | Payment confirmed |
| PaymentFailed | Payment failed |
| DispatchQueued | Invites queued for sending |
| Dispatching | Sending in progress |
| Dispatched | Sending completed |
| PartiallyDispatched | Some sends failed |
| Cancelled | Campaign cancelled |
| Refunded / PartiallyRefunded | See §14.3 |

### 4.7.4 Post-Payment Changes: Add Guests, Resend, Edit

Inviters always forget someone; support it natively from the campaign dashboard:

1. **Add guests after dispatch** — upload a delta Excel or add rows manually. Unused prepaid capacity is consumed first; beyond that, a **top-up checkout** at the campaign's per-block rate (no second $5 minimum — the minimum applies once per campaign).
2. **Resend** — per-guest "Resend" button, free, max 3 resends per guest per 24h. Resends reuse the existing token (old links keep working).
3. **Fix a guest** — edit email/phone/name/role any time; if contact info changes after a send, offer resend to the corrected address.
4. **Edit content after dispatch** — allowed; invite pages render live data, so sent links show updated details automatically. Editor banner: "This campaign is live — changes appear to guests immediately." Optional "notify guests of change" = a resend.
5. Payments carry `Kind = Initial | TopUp`; campaigns track `PaidInviteCapacity` vs `GuestCount`.

---

## 4.8 Delivery Channels

### 4.8.1 Supported Channels

| Channel | MVP Priority | Notes |
|---|---:|---|
| Email | 1 | First and easiest |
| Direct link | 1 | Copy/share manually |
| Telegram | 2 | Easier than WhatsApp if bot-based |
| WhatsApp | 3 | Requires Business API / approved templates |
| Viber | 3 | Requires bot/business setup |
| SMS | 4 | Costly; used for OTP and fallback |

### 4.8.2 Delivery Message

```text
You have a new invite from {{inviter.name}}.
Open it here: {{invite.link}}
```

Chat platforms with buttons: same text + `[Open Invite]` button.

Every delivery message footer includes: "Sent via invites.blog on behalf of {{inviter.name}} · Privacy · Remove my data" (§15).

### 4.8.3 Secure Invite Links

```text
https://me.invites.blog/i/{secure-token}
```

Token requirements: 256-bit random, unguessable, tied to one recipient, contains no database IDs, expires only if campaign settings require it. Only the token hash is stored (§9.3).

### 4.8.4 Delivery Provider Abstraction

```csharp
public interface IInviteDeliveryProvider
{
    string Channel { get; }
    Task<DeliveryResult> SendAsync(InviteDeliveryMessage message, CancellationToken cancellationToken);
}
```

Implementations: `EmailInviteDeliveryProvider`, `WhatsAppInviteDeliveryProvider`, `ViberInviteDeliveryProvider`, `TelegramInviteDeliveryProvider`, `SmsInviteDeliveryProvider`.

---

## 4.9 Invitee Experience: `me.invites.blog`

### 4.9.1 Principle: the link is the key; OTP is optional armor

| Access path | Auth required |
|---|---|
| Open `me.invites.blog/i/:token` | **None** — the 256-bit token *is* the security |
| RSVP from a token link | **None** — RSVP binds to the invite the token resolves to |
| "See all my invites" (inbox) | OTP (phone or email) |
| Invite marked *sensitive* by the inviter | OTP before viewing |

A guest can receive, view, and RSVP with **zero login** — the core happy path costs no SMS. OTP is only triggered by the inbox or sensitive campaigns.

### 4.9.2 OTP Login Flow (inbox)

1. User visits `me.invites.blog` → chooses phone or email.
2. Phone: country picker + national number → SMS OTP. Email: address → email OTP.
3. Enter 6-digit code → verified → inbox shows all invites matched to the verified `phone_e164` or email.

Soft nudge: after viewing via token, offer "Save this to your inbox" → one OTP links the invite to their identity permanently. Optional, never blocking.

### 4.9.3 Invite Link Flow

1. Guest clicks unique link.
2. Token valid + invite not sensitive → show invite directly.
3. Sensitive → request OTP first; after OTP, link invite to verified identity, then show it.

### 4.9.4 Inbox UI

Cards show: event title, inviter name, event date, venue type, RSVP status, animated thumbnail, "New" badge, "Opening soon" badge, "Past event" state.

### 4.9.5 Invite Viewing UI

Each invite opens as a full scroll-driven animated page (engine in §5): envelope appears → opens as the user scrolls → sections and colors reveal progressively → RSVP near the end → venue/map last. Must respect `prefers-reduced-motion`.

### 4.9.6 RSVP

Statuses: Going, Maybe, Not Going, Viewed only, No response.
Optional fields: guest count, meal preference, comment, arrival time, contact note.

---

# 5. Template Rendering Engine

## 5.1 Decision

Each template is a **self-contained HTML/CSS/JS package** rendered as a scroll-driven animated page. All executable code is **platform-generated**: platform designers and community designers alike author templates *declaratively* (§6); a compiler emits the package. Inviters fill in content; they never write code.

## 5.2 Template Package Structure

```text
templates/{slug}@{version}/
  index.html        # Markup with data-bound slots and conditional blocks
  styles.css        # All styling, including scroll animation keyframes
  template.js       # Scroll orchestration, envelope-open sequence
  manifest.json     # Declares variables, roles, gender variants, editable areas
  assets/
    envelope.webp
    florals.svg
```

`manifest.json` is the contract between template and platform:

```json
{
  "slug": "golden-envelope",
  "version": "1.2.0",
  "variables": ["event.title", "event.date", "venue.name", "guest.name"],
  "roles": ["bridesmaid", "groomsman", "vip", "family"],
  "genderVariants": ["male", "female", "neutral"],
  "editableAreas": ["title", "subtitle", "description", "schedule", "dressCode", "media"],
  "contentBlocks": ["bridesmaidInstructions", "maleDressCode", "femaleDressCode"]
}
```

## 5.3 Data Injection

The invite page loads the template inside a **sandboxed iframe** (`sandbox="allow-scripts"`, strict CSP, served from `assets.invites.blog`) and injects one JSON payload:

```html
<script id="invite-data" type="application/json">
  { "event": {}, "guest": {}, "venue": {}, "inviter": {}, "rsvp": {}, "theme": {} }
</script>
```

- `template.js` populates `data-var="event.title"` slots via `textContent` (never `innerHTML`) — guest/inviter content is inert by construction.
- Conditional blocks are elements tagged `data-block="bridesmaidInstructions"`; the campaign's JSON rules (§12) are evaluated **server-side**, and only the resolved block list ships to the client. Templates never evaluate rules.

## 5.4 Scroll Animation Requirements

- Primary metaphor: envelope opens as the user scrolls; sections and colors reveal progressively; RSVP near the end.
- Implementation order of preference:
  1. **CSS scroll-driven animations** (`animation-timeline: scroll()/view()`) where supported.
  2. `IntersectionObserver` + CSS class toggles as the universal fallback.
  3. A scroll library (e.g. GSAP ScrollTrigger) only where timeline scrubbing is truly needed.
- Every template ships a `prefers-reduced-motion` variant: no scrubbing, simple fades, all content reachable.
- Progressive enhancement: fully readable with JS disabled or failed — content in HTML, animation on top.
- Budget: ≤ 300KB critical path (HTML+CSS+JS); images lazy-loaded; LCP < 2s on 4G.

## 5.5 One Pipeline Everywhere

The editor's live preview, the designer's canvas preview, the guest-facing invite page, and thumbnail generation (headless screenshot) all use the **same iframe + injector**. "Preview as bridesmaid" swaps the injected `guest` object. What you preview is exactly what guests get.

## 5.6 Versioning

Templates are versioned (`golden-envelope@1.2.0`). Campaigns pin the version they were created with; template updates never silently change already-sent invites.

---

# 6. Template Designer & Community Submissions

## 6.1 Principle

Users can design fully custom templates — **declaratively, never with raw JS**. The designer produces a JSON *scene description* (`SceneJson`); the platform compiler turns it into the trusted package of §5. All executable code stays platform-generated, so community templates are safe by construction, and submission review is content review, not code review.

The `SceneJson` compiler is also the rendering layer for **platform** templates, so there is one template system, not two.

## 6.2 Designer Capabilities

**Structure**
- Section-based canvas: envelope/cover, hero, story, schedule, dress code, gallery, map/venue, role blocks, gender blocks, custom text sections, RSVP.
- Drag to reorder; add/remove/duplicate sections; per-section visibility rules (role/gender/always).

**Visual design**
- Theme: color palette with contrast checking, font pairs from a curated library, spacing scale, background textures/gradients.
- Per-section layout variants (e.g. hero: centered / split / full-bleed image).
- Asset uploads: images, SVG ornaments, logos — size/type-validated, re-encoded server-side, EXIF stripped.

**Animation (scroll-driven, preset-based)**
- Envelope opening styles: flap lift, wax-seal break, slide-out card, fold-out.
- Per-section reveal presets: fade-up, curtain, parallax layer, color wash, letter-by-letter title, ken-burns image.
- Timing controls: scroll range per animation, easing picker, stagger.
- Global intensity slider (subtle → dramatic).
- Auto-generated `prefers-reduced-motion` variant; the designer must preview it before submitting.

**Data & personalization**
- Insert any §4.2.2 variable into text; define custom variables mapped to extra Excel columns.
- Define role/gender content blocks visually — same JSON rules engine (§12).
- Live preview identical to §5.5, including role/gender preview modes.

**Guardrails**
- Compiler enforces the §5.4 budget, lazy-loads media, and rejects over-budget scenes with clear feedback.
- No raw HTML/JS input anywhere. Custom CSS: not in MVP; later possible for verified designers with server-side sanitization.

## 6.3 Designer Data Model

```csharp
public sealed class CustomTemplate
{
    public Guid Id { get; set; }
    public Guid InviterId { get; set; }
    public string Name { get; set; } = default!;
    public string SceneJson { get; set; } = default!;        // declarative design
    public string CompilerVersion { get; set; } = default!;  // compiler version pin
    public CustomTemplateStatus Status { get; set; }          // Private | Submitted | InReview | Published | Rejected | Delisted
    public Guid? PublishedTemplateId { get; set; }            // gallery template once approved
    public DateTimeOffset CreatedAt { get; set; }
    public DateTimeOffset UpdatedAt { get; set; }
}
```

## 6.4 Submission Flow

1. When the designer finishes, the checkout step offers: **"Love your design? Share it in the template gallery and get 50% off this campaign."**
2. If accepted: name the template, pick a category, confirm ownership of uploaded assets, grant the platform a non-exclusive publishing license (revocable by delisting). Attribution: "Designed by {name}" (anonymity optional).
3. Preview image + animation thumbnail auto-generated (headless render).
4. **Admin review queue** before publication: content moderation (assets, text), quality bar, performance budget re-check, duplicate detection.
5. Approved → published as a normal versioned gallery template. Rejected → clear reason; the discount stays honored (submission was in good faith; review is the platform's gate).

## 6.5 Designer Discount

- Submitting a design halves the per-invite rate for that campaign: extra blocks price at **$1 per 20 invites** instead of $1 per 10.
- The $5 minimum still applies (it covers payment processing fees).
- Granted at submission (not approval); shown at checkout as a line item: "Community template discount −50%".
- Retention lever (Should-Have): while the template remains published, the designer keeps the 50% rate on future campaigns.
- Anti-abuse: one discount per campaign regardless of submission count; near-duplicate scenes auto-flagged in review; repeat low-effort submitters lose eligibility.

## 6.6 Phasing

- **MVP:** template editor only (§4.3) on platform templates. Build the `SceneJson` compiler as the platform-template rendering layer from day one.
- **Phase A (post-MVP):** designer UI — sections, themes, animation presets — on a base "blank" family.
- **Phase B:** submission flow + review queue + discount.

---

# 7. Technical Architecture

## 7.1 Stack

### Frontend
- Angular 22, standalone components, Angular Router, Signals for state.
- HttpClient via provider-based setup; optional `httpResource` for reactive data fetching.
- SCSS or Tailwind CSS.
- CSS scroll-driven animations / WAAPI / IntersectionObserver for motion.

### Backend
- .NET 10, ASP.NET Core 10.
- Minimal APIs, vertical-slice endpoints grouped by feature.
- Entity Framework Core 10, PostgreSQL 17+.
- FluentValidation or .NET 10 Minimal API validation.
- Background jobs: Hangfire, Quartz.NET, or hosted workers.
- OpenAPI/Scalar/Swagger.

### Infrastructure
- Docker Compose for local dev.
- Nginx or Caddy reverse proxy.
- Object storage: S3-compatible, MinIO locally.
- Redis for OTP/rate limiting/queues.
- CDN for template packages (`assets.invites.blog`).

## 7.2 Solution Structure

```text
invites-blog/
  apps/
    web-inviter/                  # Angular 22 app for invites.blog
    web-invitee/                  # Angular 22 app for me.invites.blog
  api/
    InvitesBlog.Api/              # ASP.NET Core .NET 10 API
    InvitesBlog.Application/      # Use cases, services, DTOs
    InvitesBlog.Domain/           # Entities, value objects, domain events
    InvitesBlog.Infrastructure/   # EF Core, delivery providers, storage, payments
    InvitesBlog.TemplateCompiler/ # SceneJson → template package compiler
    InvitesBlog.Worker/           # Background dispatch jobs
    InvitesBlog.Tests/            # Unit/integration tests
  docs/
    spec.md
    guide-template.xlsx
  deploy/
    docker-compose.yml
    nginx.conf
```

Share a design-system library between the two Angular apps (single workspace/monorepo).

## 7.3 Frontend Applications

### `web-inviter` — `https://invites.blog`
Landing page, template browser, template editor, template designer, guest upload, validation review, venue setup, inviter details, payment checkout, campaign dashboard, guide page.

### `web-invitee` — `https://me.invites.blog`
Token invite page, optional OTP login (phone/email), invite inbox, invite detail page, RSVP, profile-lite settings, "remove my data" page.

## 7.4 Angular Patterns

Use: standalone components, route-level lazy loading, Signals for local UI state, services for API access, functional route guards and interceptors, strongly typed DTOs, component-scoped styles.

Avoid: giant NgModules, god services, thousand-line components.

### Routing: Inviter App

```text
/
/templates
/templates/:slug
/design                          # template designer (post-MVP)
/create/:templateSlug
/create/:campaignId/editor
/create/:campaignId/guests
/create/:campaignId/guests/review
/create/:campaignId/venue
/create/:campaignId/inviter
/create/:campaignId/delivery
/create/:campaignId/payment
/create/:campaignId/success
/dashboard/:campaignId           # via magic link
/guide
/pricing
/privacy
/terms
```

### Routing: Invitee App

```text
/
/login
/verify
/inbox
/i/:token
/invites/:inviteId
/invites/:inviteId/rsvp
/privacy/remove/:token           # guest data removal
```

### App Config Example

```ts
import { ApplicationConfig } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideHttpClient, withInterceptors } from '@angular/common/http';
import { routes } from './app.routes';
import { authInterceptor } from './core/interceptors/auth.interceptor';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideHttpClient(withInterceptors([authInterceptor]))
  ]
};
```

---

# 8. Backend Architecture & Entities

## 8.1 API Project Structure

```text
InvitesBlog.Api/
  Program.cs
  Endpoints/
    TemplatesEndpoints.cs
    DesignerEndpoints.cs
    CampaignsEndpoints.cs
    GuestsEndpoints.cs
    PaymentsEndpoints.cs
    DeliveryEndpoints.cs
    OtpEndpoints.cs
    InboxEndpoints.cs
    RsvpEndpoints.cs
    PrivacyEndpoints.cs
  Middleware/
  Filters/

InvitesBlog.Application/
  Templates/ Designer/ Campaigns/ Guests/ Payments/ Delivery/ Otp/ Inbox/ Rsvp/ Privacy/

InvitesBlog.Domain/
  Entities/ ValueObjects/ Enums/ Events/

InvitesBlog.Infrastructure/
  Persistence/ Storage/ Payments/ Delivery/ Otp/ Security/
```

## 8.2 Main Entities

### Template

```csharp
public sealed class Template
{
    public Guid Id { get; set; }
    public string Name { get; set; } = default!;
    public string Slug { get; set; } = default!;
    public string Version { get; set; } = default!;
    public string Category { get; set; } = default!;
    public string Description { get; set; } = default!;
    public string PreviewImageUrl { get; set; } = default!;
    public string? PreviewAnimationUrl { get; set; }
    public bool IsPremium { get; set; }
    public Guid? DesignerInviterId { get; set; }   // community attribution
    public string SceneJson { get; set; } = default!;
    public string ManifestJson { get; set; } = default!;
    public string PackageUrl { get; set; } = default!;   // compiled package on assets CDN
    public bool IsActive { get; set; } = true;
    public DateTimeOffset CreatedAt { get; set; }
}
```

### Inviter

```csharp
public sealed class Inviter
{
    public Guid Id { get; set; }
    public string Name { get; set; } = default!;
    public string PhoneE164 { get; set; } = default!;
    public string Email { get; set; } = default!;       // unique, lowercased
    public string? Organization { get; set; }
    public DateTimeOffset CreatedAt { get; set; }
}
```

Inviters are deduplicated by normalized email: a returning inviter reuses their row, enabling future multi-campaign continuity with zero migration.

### Campaign

```csharp
public sealed class Campaign
{
    public Guid Id { get; set; }
    public Guid TemplateId { get; set; }
    public string TemplateVersion { get; set; } = default!;   // pinned
    public Guid? InviterId { get; set; }                       // set at inviter-details step
    public string AccessTokenHash { get; set; } = default!;   // §4.6.2 possession token
    public string Title { get; set; } = default!;
    public string Slug { get; set; } = default!;
    public CampaignStatus Status { get; set; }
    public string EventType { get; set; } = default!;
    public DateTimeOffset EventStartAt { get; set; }
    public DateTimeOffset? EventEndAt { get; set; }
    public int PaidInviteCapacity { get; set; }
    public bool HasDesignerDiscount { get; set; }
    public int RetentionDays { get; set; } = 90;              // §15.4
    public string CustomContentJson { get; set; } = default!;
    public string ThemeOverridesJson { get; set; } = default!;
    public DateTimeOffset CreatedAt { get; set; }
    public DateTimeOffset UpdatedAt { get; set; }
}
```

### Guest

```csharp
public sealed class Guest
{
    public Guid Id { get; set; }
    public Guid CampaignId { get; set; }
    public string? Email { get; set; }
    public string? PhoneE164 { get; set; }
    public string? PhoneRaw { get; set; }
    public string Name { get; set; } = "Guest";
    public string? Role { get; set; }
    public string Gender { get; set; } = "unspecified";
    public string MetadataJson { get; set; } = "{}";
    public bool OptedOut { get; set; }                 // §15.3
    public DateTimeOffset CreatedAt { get; set; }
}
```

### Invite

```csharp
public sealed class Invite
{
    public Guid Id { get; set; }
    public Guid CampaignId { get; set; }
    public Guid GuestId { get; set; }
    public string TokenHash { get; set; } = default!;
    public bool RequiresOtp { get; set; }              // sensitive campaigns
    public InviteStatus Status { get; set; }
    public RsvpStatus RsvpStatus { get; set; }
    public DateTimeOffset? ViewedAt { get; set; }
    public DateTimeOffset? RespondedAt { get; set; }
    public DateTimeOffset CreatedAt { get; set; }
}
```

### DeliveryAttempt

```csharp
public sealed class DeliveryAttempt
{
    public Guid Id { get; set; }
    public Guid InviteId { get; set; }
    public string Channel { get; set; } = default!;
    public string RecipientAddress { get; set; } = default!;
    public DeliveryStatus Status { get; set; }
    public string? ProviderMessageId { get; set; }
    public string? ErrorMessage { get; set; }
    public DateTimeOffset AttemptedAt { get; set; }
}
```

### Payment

```csharp
public sealed class Payment
{
    public Guid Id { get; set; }
    public Guid CampaignId { get; set; }
    public PaymentKind Kind { get; set; }              // Initial | TopUp
    public int InviteCount { get; set; }
    public decimal Amount { get; set; }
    public string Currency { get; set; } = "USD";
    public PaymentStatus Status { get; set; }
    public string Provider { get; set; } = default!;
    public string? ProviderSessionId { get; set; }
    public string? ProviderPaymentId { get; set; }
    public DateTimeOffset CreatedAt { get; set; }
    public DateTimeOffset? PaidAt { get; set; }
}

public sealed class Refund
{
    public Guid Id { get; set; }
    public Guid PaymentId { get; set; }
    public decimal Amount { get; set; }
    public RefundStatus Status { get; set; }
    public string? ProviderRefundId { get; set; }
    public DateTimeOffset CreatedAt { get; set; }
}
```

### OtpChallenge

```csharp
public sealed class OtpChallenge
{
    public Guid Id { get; set; }
    public OtpChannel Channel { get; set; }            // Sms | Email
    public string? PhoneE164 { get; set; }
    public string? Email { get; set; }
    public string CodeHash { get; set; } = default!;
    public int Attempts { get; set; }
    public DateTimeOffset ExpiresAt { get; set; }
    public DateTimeOffset CreatedAt { get; set; }
    public DateTimeOffset? VerifiedAt { get; set; }
}
```

---

# 9. Database Design

## 9.1 Tables

```text
templates
custom_templates
inviters
campaigns
guests
invites
delivery_attempts
payments
refunds
otp_challenges
rsvp_responses
uploaded_guest_files
template_assets
campaign_assets
suppression_list
audit_logs
```

## 9.2 Important Indexes

```sql
CREATE INDEX idx_templates_slug ON templates(slug);
CREATE UNIQUE INDEX idx_inviters_email ON inviters(lower(email));
CREATE INDEX idx_campaigns_status ON campaigns(status);
CREATE UNIQUE INDEX idx_campaigns_access_token_hash ON campaigns(access_token_hash);
CREATE INDEX idx_guests_campaign_id ON guests(campaign_id);
CREATE INDEX idx_guests_phone_e164 ON guests(phone_e164);
CREATE INDEX idx_guests_email ON guests(email);
CREATE INDEX idx_invites_campaign_id ON invites(campaign_id);
CREATE UNIQUE INDEX idx_invites_token_hash ON invites(token_hash);
CREATE INDEX idx_delivery_invite_id ON delivery_attempts(invite_id);
CREATE INDEX idx_otp_phone_expires ON otp_challenges(phone_e164, expires_at);
CREATE INDEX idx_otp_email_expires ON otp_challenges(email, expires_at);
CREATE UNIQUE INDEX idx_suppression_contact ON suppression_list(contact_hash);
```

## 9.3 Token Storage

Never store raw tokens (invite tokens, campaign access tokens, dashboard tokens). Store a SHA-256 or HMAC hash. On use: read token from URL/header → hash → match → authorize.

---

# 10. API Specification

Auth legend: **[campaign-token]** = campaign access token Bearer (§4.6.2); **[invitee-jwt]** = OTP-issued JWT; **[admin]** = admin auth; **(public)** = no auth.

## 10.1 Templates

- `GET /api/templates` (public) — active templates. Query: `category`, `search`, `page`, `pageSize`.
- `GET /api/templates/{slug}` (public) — template details.
- `POST /api/admin/templates` [admin] — create platform template.

## 10.2 Designer (post-MVP)

- `POST /api/designer/templates` [campaign-token or designer session] — create draft `CustomTemplate`.
- `PUT /api/designer/templates/{id}/scene` — save `SceneJson`.
- `POST /api/designer/templates/{id}/compile-preview` — compile and return preview package URL.
- `POST /api/designer/templates/{id}/submit` — submit to gallery; grants campaign discount.
- `GET /api/admin/designer/submissions` [admin] — review queue; approve/reject endpoints.

## 10.3 Campaign Builder

- `POST /api/campaigns` (public) — create draft from template.

  ```json
  { "templateId": "uuid", "title": "Aisha & Adam Wedding" }
  ```

  Response:

  ```json
  { "campaignId": "uuid", "status": "Draft", "accessToken": "opaque-256-bit" }
  ```

- `PUT /api/campaigns/{id}/content` [campaign-token]
- `PUT /api/campaigns/{id}/venue` [campaign-token]
- `PUT /api/campaigns/{id}/inviter` [campaign-token] — also triggers the "Resume your invite" magic-link email.
- `PUT /api/campaigns/{id}/delivery-settings` [campaign-token]

  ```json
  {
    "channels": ["email", "whatsapp"],
    "fallbackChannel": "email",
    "messageTemplate": "You have a new invite from {{inviter.name}}. Open it here: {{invite.link}}"
  }
  ```

- `GET /api/campaigns/{id}/summary` [campaign-token]
- `POST /api/campaigns/access/resend-link` (public) — email me my resume/dashboard link (never displays it). Rate limited.

## 10.4 Guest Upload

- `POST /api/campaigns/{id}/guests/upload` [campaign-token] — multipart; includes `defaultCountry`.

  ```json
  {
    "uploadId": "uuid",
    "totalRows": 120,
    "validRows": 115,
    "invalidRows": 5,
    "duplicates": 2,
    "warnings": ["3 phone numbers may have been stored as scientific notation."],
    "errors": [{ "row": 9, "field": "email", "message": "Invalid email address." }]
  }
  ```

- `POST /api/campaigns/{id}/guests/confirm-upload` [campaign-token]
- `POST /api/campaigns/{id}/guests` [campaign-token] — add guests after payment (delta upload or manual rows).
- `PUT /api/campaigns/{id}/guests/{guestId}` [campaign-token] — fix contact details.
- `POST /api/campaigns/{id}/guests/{guestId}/resend` [campaign-token] — free, rate-limited resend.

## 10.5 Payment

- `POST /api/campaigns/{id}/checkout` [campaign-token] → `{ "checkoutUrl": "..." }`
- `POST /api/campaigns/{id}/topup` [campaign-token] — checkout for added capacity.
- `POST /api/payments/webhook` (provider-signed) — idempotent. On success: mark payment paid → mark campaign paid → queue dispatch (initial payments only).
- `POST /api/campaigns/{id}/cancel` [campaign-token] — pre-dispatch: auto full refund; post-dispatch: cancel + graceful invite pages (§14.3).

## 10.6 Dispatch

- `POST /api/campaigns/{id}/dispatch` [admin/system] — queues delivery jobs; normally triggered by payment webhook.

## 10.7 OTP

- `POST /api/otp/request` (public, rate-limited, CAPTCHA after 2nd request)

  ```json
  { "channel": "sms", "phone": "+9607777777" }
  ```

  or `{ "channel": "email", "email": "aisha@example.com" }`

  Response: `{ "challengeId": "uuid", "expiresInSeconds": 300 }`

- `POST /api/otp/verify` (public)

  ```json
  { "challengeId": "uuid", "code": "123456" }
  ```

  Response: `{ "accessToken": "jwt", "refreshToken": "opaque" }`

## 10.8 Invitee

- `GET /api/invites/by-token/{token}` (public) — resolves token; returns invite render data, or `{ "requiresOtp": true }` for sensitive invites.
- `POST /api/invites/by-token/{token}/rsvp` (public) — RSVP without login.

  ```json
  { "status": "Going", "guestCount": 1, "comment": "Looking forward to it." }
  ```

- `GET /api/me/invites` [invitee-jwt] — inbox: invites matched to verified phone/email.
- `POST /api/invites/{inviteId}/claim` [invitee-jwt] — "save to my inbox" after token view.

## 10.9 Privacy

- `GET /api/privacy/remove/{token}` (public) — guest data-removal page data.
- `POST /api/privacy/remove/{token}` (public) — anonymize guest, add contact to suppression list.
- `DELETE /api/campaigns/{id}` [campaign-token] — inviter hard-deletes campaign + guest data (audited).

---

# 11. Authentication & Security

## 11.1 Invitee OTP

- 6-digit code; expiry 5 minutes; max 5 attempts.
- Channels: SMS and email (both in MVP).
- Rate limits: 3 sends per contact per hour, 10 per day; per-IP limits; CAPTCHA after the 2nd request.
- Store only hashed OTP; never log codes.
- OTP SMS spend tracked separately from delivery SMS in admin dashboard.

## 11.2 Inviter Access (no accounts)

- Campaign access token: 256-bit random, hash stored, Bearer on all campaign writes; campaign IDs alone grant nothing.
- "Resume your invite" magic link emailed at inviter-details step.
- Post-payment dashboard magic link: long-lived, campaign-scoped, revocable, regenerable via email-only form.

## 11.3 Invite Token Security

- 256-bit random; hash stored only; no IDs embedded; optional expiry per campaign; optional OTP-before-view for sensitive campaigns.

## 11.4 Template Sandbox

- Template packages served from `assets.invites.blog` under strict CSP, rendered in `sandbox="allow-scripts"` iframes; templates cannot access the parent app, cookies, or tokens.
- Guest data injected as JSON and applied via `textContent`; server-side rule resolution (§5.3).

## 11.5 Abuse Prevention

- CAPTCHA on campaign creation and OTP request.
- File upload size limits.
- Rate limits on OTP, resends, dispatch, and link-resend forms.
- Delivery throttling.
- Admin review for suspicious campaigns; blocklist phone/email support (E.164-compared).
- Suppression list honored on every upload (§15.3).

---

# 12. Dynamic Personalization Engine

## 12.1 Goal

Render different invite content based on guest metadata: name, email, phone, role, gender, custom metadata.

## 12.2 Rules Format

Rules are JSON, evaluated server-side, never executable code:

```json
{
  "rules": [
    { "condition": { "field": "role",   "operator": "equals", "value": "bridesmaid" }, "contentBlock": "bridesmaidInstructions" },
    { "condition": { "field": "gender", "operator": "equals", "value": "male" },       "contentBlock": "maleDressCode" }
  ]
}
```

## 12.3 Rendering Input

```json
{ "event": {}, "inviter": {}, "guest": {}, "venue": {}, "rsvp": {}, "theme": {} }
```

## 12.4 Rendering Safety

- Escape user content by default; sanitize rich text.
- No arbitrary JavaScript in templates (all JS is compiler-generated).
- Only approved components and fields.
- Every conditional block has neutral default content so `unspecified` guests see a complete invite.

---

# 13. Delivery System

## 13.1 Dispatch Flow

1. Payment success webhook received (idempotent).
2. Campaign marked paid; dispatch job created.
3. For each invitee: generate secure token → render delivery message → pick channel → send → store delivery attempt.
4. Campaign status updated (Dispatched / PartiallyDispatched).

## 13.2 Fallback Logic

```text
If WhatsApp fails and email exists → send email.
If Viber fails and phone exists → SMS fallback if enabled.
If all fail → mark failed, show in campaign report, auto-credit capacity (§14.3).
```

## 13.3 Delivery Report (campaign dashboard)

Total invites, sent, failed, viewed, RSVP counts (Going/Maybe/Not Going), channel breakdown, per-guest status with Resend buttons.

---

# 14. Payment Implementation

## 14.1 Provider Interface

```csharp
public interface IPaymentProvider
{
    Task<CheckoutSessionResult> CreateCheckoutSessionAsync(CreateCheckoutSessionRequest request, CancellationToken cancellationToken);
    Task<PaymentWebhookResult> HandleWebhookAsync(HttpRequest request, CancellationToken cancellationToken);
    Task<RefundResult> RefundAsync(RefundRequest request, CancellationToken cancellationToken);
}
```

## 14.2 Supported Providers

MVP: Stripe (international) or local gateway (Maldives-first). Future: BML, Ooredoo Pay, bank transfer with manual confirmation.

## 14.3 Cancellation & Refunds

| Situation | Policy |
|---|---|
| Paid, **not yet dispatched** | Self-service cancel → automatic full refund via provider API |
| **Partially dispatched** (provider failures) | Failed sends auto-credit capacity; unused-capacity refund via support |
| Fully dispatched | No refund (service delivered); cancelling shows a graceful "This event was cancelled" page on all invite links |
| Event cancelled by inviter | One free cancellation-notice blast to all guests |

Refund webhooks are idempotent like payment webhooks. `Refunded`/`PartiallyRefunded` payment states; `Refund` rows link to the original `Payment`.

---

# 15. Privacy & Data Protection

Guests never consented — the inviter uploaded them. Handle this explicitly:

## 15.1 Roles

The inviter is data controller for their guest list; invites.blog is processor (and controller for its own operations). Stated in Terms plus a short DPA-style clause accepted at upload.

## 15.2 Transparency

Every invite page and delivery message footer: "Sent via invites.blog on behalf of {{inviter.name}} · Privacy · Remove my data."

## 15.3 Guest Self-Service Removal

The "Remove my data" link (tokenized, no login) lets a guest delete their contact data: the invite is anonymized, the inviter sees "guest opted out" in the report, and the contact joins a **suppression list** (hashed) so future uploads by any inviter cannot re-message them without their action.

## 15.4 Retention

Campaign guest data auto-deletes **90 days after the event date** (per-campaign configurable, min 30 / max 365). Delivery logs keep only hashed addresses afterward. Marketed as a feature: "We don't keep your guest list forever."

## 15.5 Inviter Deletion

Dashboard "Delete campaign & all guest data" — immediate hard delete plus audit log entry.

## 15.6 Minimization

Guest contacts are never used for marketing; delivery messages contain no tracking beyond the invite-view event itself. Admin actions are audited. Invite tokens never leak other invitees; guests only ever see their own personalized invite.

---

# 16. Admin Features

## 16.1 Admin Dashboard

Manage: templates, community submissions (review queue), campaigns, payments/refunds, delivery attempts, OTP spend, abuse reports, suppression list, inviters, platform settings.

## 16.2 Template Management

Create/publish/unpublish platform templates, upload assets, define fields and role/gender blocks — all through the same `SceneJson` pipeline used by the designer.

---

# 17. MVP Scope

## 17.1 MVP Must-Have

1. `invites.blog` landing page.
2. Template gallery + detail preview.
3. `SceneJson` compiler rendering platform templates (§5, §6.6).
4. Template editor.
5. Excel upload, E.164 normalization, validation, review screen.
6. Venue/event type form.
7. Inviter details form + campaign access token + resume magic link.
8. Pricing calculation + payment checkout + idempotent webhook.
9. Email delivery.
10. Secure invite token links; token-only viewing + RSVP with zero login.
11. Phone **and email** OTP; invite inbox.
12. Campaign dashboard magic link with delivery/RSVP report.
13. Guest "remove my data" link + suppression list.
14. Guide page.

## 17.2 MVP Should-Have

1. Direct share link.
2. Role-based and gender-based rendering.
3. Add-guests top-up + per-guest resend.
4. Pre-dispatch self-service cancel/refund.
5. Retention auto-delete job.
6. Download invalid-rows report.

## 17.3 Post-MVP

1. Template designer (Phase A) and community submissions + discount (Phase B).
2. Telegram, WhatsApp, Viber delivery; SMS fallback.
3. WhatsApp/Telegram OTP.
4. Inviter multi-campaign view (email OTP continuity).
5. Custom domains, advanced analytics, AI template text suggestions, designer marketplace/revenue share.

---

# 18. Development Milestones

**Phase 1 — Foundation:** repo, Angular 22 apps, .NET 10 API, PostgreSQL, Docker Compose, CI, design system.
**Phase 2 — Template Engine + Magazine:** `SceneJson` compiler, sandboxed renderer, landing page, template grid/preview, seeded templates.
**Phase 3 — Campaign Builder:** create campaign + access token, editor, venue, inviter details, resume magic link, save draft.
**Phase 4 — Guest Upload:** Excel parser, E.164 normalization, validation, review screen, confirm, guide page.
**Phase 5 — Payment:** pricing engine, checkout, idempotent webhook, payment states.
**Phase 6 — Delivery:** token generation, email provider, dispatch worker, delivery attempts, fallback.
**Phase 7 — Invitee Experience:** token invite route (no login), animated viewer, RSVP, phone+email OTP, inbox, claim-to-inbox.
**Phase 8 — Dashboard + Hardening:** campaign dashboard, add-guests/resend, refunds, privacy removal + suppression, retention job, admin, rate limits, monitoring.
**Phase 9 (post-MVP) — Designer:** designer UI, submission flow, review queue, discount.

---

# 19. Non-Functional Requirements

## 19.1 Performance
- Landing page LCP < 2.5s on good mobile connection; invite page < 2s after first request.
- Template package critical path ≤ 300KB; previews and media lazy-loaded; CDN for static assets.

## 19.2 Accessibility
- `prefers-reduced-motion` respected by every template (auto-generated variant).
- Keyboard navigation, semantic HTML, sufficient contrast (enforced in designer palette), screen-reader labels for RSVP and forms.
- Invites fully readable without JS.

## 19.3 Reliability
- Dispatch jobs retryable; payment and refund webhooks idempotent; delivery attempts logged; OTP retries handled safely.

## 19.4 Privacy
- No public guest lists; tokens leak nothing; guests see only their own invite; admin actions audited. Full policy in §15.

## 19.5 Compliance
- GDPR-style handling (§15), consent for any marketing messages, retention rules, deletion flows, delivery-provider terms compliance.

---

# 20. Docker Compose Local Development

```yaml
services:
  postgres:
    image: postgres:17
    environment:
      POSTGRES_DB: invites_blog
      POSTGRES_USER: invites
      POSTGRES_PASSWORD: invites_password
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7
    ports:
      - "6379:6379"

  minio:
    image: minio/minio
    command: server /data --console-address ":9001"
    environment:
      MINIO_ROOT_USER: minio
      MINIO_ROOT_PASSWORD: minio_password
    ports:
      - "9000:9000"
      - "9001:9001"
    volumes:
      - minio_data:/data

volumes:
  postgres_data:
  minio_data:
```

---

# 21. Environment Variables

## API

```text
ASPNETCORE_ENVIRONMENT=Development
ConnectionStrings__Postgres=Host=localhost;Port=5432;Database=invites_blog;Username=invites;Password=invites_password
Jwt__Issuer=invites.blog
Jwt__Audience=invites.blog
Jwt__SigningKey=change-this-in-production
Storage__Provider=Minio
Storage__Bucket=invites-assets
Email__Provider=Resend
Email__ApiKey=...
Payments__Provider=Stripe
Payments__SecretKey=...
Otp__SmsProvider=SmsProvider
Otp__EmailProvider=Resend
Otp__ExpiryMinutes=5
Privacy__DefaultRetentionDays=90
```

## Angular

```text
API_BASE_URL=https://api.invites.blog
ASSETS_BASE_URL=https://assets.invites.blog
INVITEE_BASE_URL=https://me.invites.blog
INVITER_BASE_URL=https://invites.blog
```

---

# 22. Testing Strategy

## 22.1 Backend
- Unit: pricing (incl. designer discount + top-ups), rule evaluation, token generation/hash lookup, E.164 normalization.
- Integration: guest upload, payment + refund webhook idempotency, dispatch retries/fallback, suppression-list enforcement, retention job.
- Security: invite token access, campaign-token authorization on every write endpoint, sensitive-invite OTP gate.

## 22.2 Frontend
- Template selection and builder flows, guest upload validation screen, checkout, token invite page (no login), OTP login (phone + email), inbox, RSVP, reduced-motion behavior, JS-disabled invite readability.

## 22.3 Manual QA Scenarios
1. Email-only guest receives invite.
2. Phone-only guest receives invite; local-format number normalizes correctly.
3. Guest with both email and phone receives invite.
4. Bridesmaid sees bridesmaid variant; groomsman sees groomsman variant.
5. Male/female guests see different dress code blocks; `unspecified` guest sees complete neutral invite.
6. Guest opens link, views, and RSVPs with zero login.
7. Guest saves invite to inbox via one OTP; inbox matches across number formats.
8. Invalid Excel upload shows clear row-level errors.
9. Payment success triggers dispatch once; duplicate webhook does not resend.
10. Inviter clears browser, recovers campaign via resume magic link.
11. Inviter adds 5 guests after dispatch; top-up charges correct amount; new invites send.
12. Pre-dispatch cancel refunds automatically.
13. Guest uses "Remove my data"; contact is suppressed on the next upload.
14. OTP rate limit and CAPTCHA trigger correctly.

---

# 23. Risks and Mitigations

## 23.1 WhatsApp/Viber Approval
Risk: business messaging requires approval, paid templates, compliance.
Mitigation: launch with email + direct links; Telegram next; WhatsApp/Viber after approval.

## 23.2 Excel Chaos
Risk: malformed spreadsheets.
Mitigation: sample file, tolerant parser, row-level errors, correction before payment.

## 23.3 Payment Fees
Risk: `$1 per 10 invites` too low after fees.
Mitigation: $5 minimum including 50 invites; minimum applies once per campaign (top-ups exempt).

## 23.4 SMS Cost
Risk: OTP SMS is a real per-login cost.
Mitigation: token-first access means the happy path needs no OTP at all; email OTP in MVP; strict send limits; WhatsApp/Telegram OTP post-MVP.

## 23.5 Privacy
Risk: invite links exposing private event data; non-consenting guest contacts.
Mitigation: 256-bit hashed tokens, optional OTP-gated viewing, no public guest lists, removal link + suppression list, retention auto-delete.

## 23.6 Animation Overload
Risk: heavy animations make invites slow.
Mitigation: compiler-enforced 300KB budget, lazy loading, reduced-motion variants, CSS transform/opacity-only animations.

## 23.7 Community Template Abuse
Risk: low-quality or infringing submissions farming the discount.
Mitigation: review queue before publication, duplicate-scene detection, one discount per campaign, eligibility revocation for repeat low-effort submitters, asset-ownership confirmation at submission.

---

# 24. Recommended First Build Order

1. Database schema.
2. `SceneJson` compiler + sandboxed template renderer (one template).
3. Templates API.
4. Landing page + template gallery.
5. Campaign creation + access token.
6. Editor with one template.
7. Excel upload + E.164 normalization.
8. Invite token generation.
9. Token invite view page (no login) + RSVP.
10. Email dispatch.
11. Payment + webhook.
12. OTP (phone + email) + inbox.
13. Campaign dashboard (report, add guests, resend).
14. Privacy removal + retention job.
15. Polish animations last, after the flow works end to end.

The compiler comes early on purpose: it is the single rendering pipeline for platform templates now and the designer later.

---

# 25. Definition of Done (MVP)

- An inviter can select a template and customize event details — with no account.
- The inviter can upload Excel guests; the system validates and normalizes them.
- The inviter can pay; the system sends invite links by email exactly once.
- Invitees can open their link, view a personalized animated invite, and RSVP with zero login.
- Invitees can optionally verify by phone or email OTP and see their inbox.
- The inviter receives a dashboard magic link showing delivery and RSVP results, and can add guests, resend, or cancel.
- A guest can remove their data via the footer link.
- The inviter can recover access to their campaign via the resume magic link.

---

# 26. Appendix A: Example User Stories

## Inviter

```text
As an inviter, I want to browse animated templates so I can choose a style that fits my event.
As an inviter, I want to create and pay for a campaign without creating an account so the process is fast.
As an inviter, I want to customize text, date, venue, and images so the invite feels personal.
As an inviter, I want to upload an Excel guest list so I do not need to enter guests manually.
As an inviter, I want different roles to receive different content so bridesmaids, groomsmen, VIPs, and general guests get relevant instructions.
As an inviter, I want to add forgotten guests and resend invites after paying so mistakes are cheap to fix.
As an inviter, I want a dashboard link in my email so I can check RSVPs any time from any device.
```

## Invitee

```text
As an invitee, I want to open my invite from a link with no login so I can see event details instantly.
As an invitee, I want to RSVP right from the invite so responding takes seconds.
As an invitee, I want to optionally verify once so all my invites live in one inbox.
As an invitee, I want the invite to unfold as I scroll so it feels special.
As an invitee, I want a way to remove my contact details so my data is respected.
```

## Designer

```text
As a designer, I want to build a custom template with sections, themes, and scroll animations without writing code.
As a designer, I want to preview my template as different guests, including reduced motion, before submitting.
As a designer, I want to submit my template to the gallery and get 50% off my campaign, with attribution on the template.
```

---

# 27. Appendix B: Excel Upload Guide Copy

Your Excel file should have the following columns in the first row:

```text
email | phone | name | role | gender
```

At least one of `email` or `phone` is required.

Good example:

| email | phone | name | role | gender |
|---|---|---|---|---|
| aisha@example.com | +9607777777 | Aisha | bridesmaid | female |
| adam@example.com | 9999999 | Adam | groomsman | male |

Phone tips: include the country code (e.g. `+960…`) or select your country when uploading and use local numbers — we'll format them for you.

Common mistakes to avoid:

- Do not merge cells.
- Do not put phone numbers in scientific notation.
- Do not put multiple guests in one row.
- Do not rename required headers.
- Do not leave both email and phone empty.

Role examples: `bridesmaid, groomsman, vip, family, speaker, guest, staff`
Gender examples: `male, female, neutral, unspecified`

---

# 28. Appendix C: Stack References

- Angular Router is the official Angular navigation library, included by default in Angular CLI projects.
- ASP.NET Core supports HTTP APIs, Minimal APIs, OpenAPI documentation, validation, and cloud-ready services.
- ASP.NET Core in .NET 10 includes validation support for Minimal APIs.
- Angular's modern architecture favors standalone components and provider-based setup for routing and HTTP.
- CSS scroll-driven animations (`animation-timeline`) with `IntersectionObserver` fallback cover the scroll-reveal requirements without heavy dependencies.
