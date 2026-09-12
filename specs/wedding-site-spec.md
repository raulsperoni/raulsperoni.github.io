# Wedding website — specification v1

Status: agreed product scope, with proposed implementation defaults called out below.

## Goal and design

A personal, playful wedding website for approximately 200 guests that is easy for the couple and planner to operate. Start with save-the-date, then collect RSVPs through the same household links. Reliability and a small maintenance burden take priority over extra features.

Preserve the visual character, mobile layout, typography, gallery, and animations of [Juho’s invitation](https://juhonamnam.github.io/wedding-invitation/). Adapt its [React source](https://github.com/juhonamnam/wedding-invitation); replace the Korean content and local integrations. Spanish is the default language, with an English switch. All guest-facing content and messages must be available in both languages. Language changes must preserve the invitation and any entered answers.

## Access and households

- Public visitors see only the couple’s names, wedding date, and an invitation-code entry field. Venue, schedule, and household information require a valid code.
- Each household has one stable invitation URL containing a memorable, randomly generated word code. The same code can be typed manually if the link is lost. Guests need no account or password.
- Anyone with the household code can view and respond for that household; this tradeoff is accepted.
- Organizers set the household’s allocated places. Each place represents either a named guest or an unnamed plus-one. Guests may fill in plus-one names but cannot add places.
- Each guest has their own attendance response. One household member can accept for themselves and decline for another.
- Organizers can replace a code if needed, invalidating the previous one.

Proposed defaults: use easy-to-type words without accents, tolerate case and surrounding spaces, rate-limit failed code attempts, and avoid exposing private details in public HTML, downloadable site data, or WhatsApp previews. The exact code length/word list is an implementation choice; codes must not be derived from guest names.

## Guest journey

| Stage | Guest experience | Recorded state |
| --- | --- | --- |
| Save-the-date | Personalized household page; date; add-to-calendar options; “¡Recibido!” / “Got it!” button | Household acknowledgment and timestamp |
| RSVP open | Same link reveals the RSVP form, event details, and deadline | Individual attendance responses, plus-one names, latest submission time |
| RSVP closed | Guests can read their saved answers and event details; changes require contacting an organizer | Existing answers retained; organizers can still edit |

- The save-the-date acknowledgment is separate from attendance. Opening a link or clicking a calendar button does not count as acknowledgment or RSVP.
- Calendar options: Google Calendar and downloadable ICS for Apple Calendar/Outlook. Use the wedding’s configured timezone; never guess missing times or venue details. A date-only save-the-date can be an all-day event.
- Updating the website does not automatically update calendar entries already downloaded by guests. Calendar links should always generate the currently published details.
- Attendance states: unanswered, attending, not attending. Household summaries distinguish pending, partially answered, and fully answered households.
- Guests can revisit and revise responses until the configured deadline. Repeated submissions update the same guest records rather than creating new guests or duplicate responses.
- Show success only after persistence succeeds. On failure, retain entered answers and make retry possible.
- Additional RSVP questions remain undecided. Dietary needs and transport are possibilities, not committed features. Do not build a generic form builder.

Proposed defaults: organizers explicitly open RSVP from the admin; a configured deadline closes guest edits automatically on the server. Organizers remain able to change that deadline. Choose whether complete household answers are required before submission during form design.

## WhatsApp communication

- Organizers send invitations and reminders personally. No automatic WhatsApp sending, messaging service, or email campaign is required.
- Each household has a copy-link action and a WhatsApp action that opens a prepared Spanish or English message containing its invitation link. The organizer reviews and presses Send.
- Support save-the-date, RSVP invitation, and reminder wording. Keep messages editable before sending.
- Do not label a message delivered or read just because WhatsApp was opened. Acknowledgment and submitted responses are the reliable guest actions.

Proposed default: optional household contact number; when absent, copy/share the message. A manual “sent” marker can help avoid sending twice.

## Organizer experience

The user, partner, and planner have separate logins and equal wedding-management permissions. All can:

- Create and edit households, guest names, allocations, and optional contact information.
- Generate, copy, and replace household links/codes.
- Manage published wedding details, language content, stage, and RSVP deadline.
- View acknowledgments, pending/partial/completed responses, and attendance totals.
- Correct answers after the guest deadline and export the guest list as CSV.
- Access uploaded photos if the optional photo feature is enabled.

Prefer Django admin with a few focused actions and useful filters. No separate custom management application is required. Retain a basic record of organizer changes so one person’s correction can be understood by the others. Confirm changes that remove already-answered guest places; never silently discard responses.

## Optional extra: QR photo dump

This is a separate milestone and must not delay save-the-date or RSVP launch.

- A QR code on tables opens a phone-friendly photo-upload page, without requiring a guest account.
- Guests can select multiple photos and see which uploads succeeded or failed.
- Only the three organizers can browse and download the collection. There is no public gallery or live slideshow.
- Upload access is separate from household RSVP access: the table QR must not grant access to anyone’s invitation or answers.
- Proposed defaults: a shared upload-only event code, uploads enabled for a configurable period, file-size/type limits, and private durable media storage. Confirm supported phone-photo formats and storage/retention before this milestone.

## Planned implementation

| Layer | Choice |
| --- | --- |
| Frontend | Juho’s React + TypeScript + Vite + SCSS |
| Backend | Django replacing the small Go backend |
| Management | Django admin |
| Database | SQLite on persistent storage |
| Deployment | One Railway service/container; Django serves the built frontend and API |
| Media | Site images bundled with the site; optional guest uploads stored privately and durably |

Use a supported Django release. Guest authorization and deadlines must be enforced by the server. Keep the database outside the deploy image, enable backups, and verify a restore before real invitations go out. Reuse MIT-licensed frontend code with its license preserved.

## Launch checks

1. Public visitors cannot retrieve private event or guest information.
2. The same household code works in a link and by manual entry; it survives the switch from save-the-date to RSVP.
3. Acknowledging save-the-date does not set attendance. Calendar files open with the correct date/time in the chosen apps.
4. A household can respond differently for individual guests, name allowed plus-ones, and never exceed its allocation.
5. Repeat submissions and edits do not duplicate guests or responses; failed saves do not show success.
6. The deadline prevents guest edits while allowing all three organizers to make corrections.
7. All three organizers can manage guests and export accurate attendance lists without developer help.
8. Spanish/English work on phones, including the browser opened from WhatsApp.
9. Responses survive a deployment and can be recovered from backup.
10. If photos ship: the QR allows uploads, never browsing or RSVP access; uploads survive redeployments.

## Still open

- Couple’s display names, date/time, venue, domain, photos, and final wording.
- RSVP deadline and any additional questions.
- Whether partial household submissions are allowed; proposed default is to preserve partial responses and clearly show who remains unanswered.
- Photo milestone timing, supported formats, storage, and retention.

Not in the first version: automated messaging, seating plans, a public photo feed, slideshow, guest accounts, or a general-purpose website/form builder.
