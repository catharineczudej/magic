# MAGIC: Rebuild Audit

*Prepared 17 Oct 2026 for a non-technical reader. No code was changed to produce this.*

---

## The one-paragraph version

MAGIC today is a **polished demo, not a working product**. The video player is good: it plays smoothly, swipes like TikTok, looks like the designs, and installs on an iPhone home screen. Google sign-in and the onboarding flow (phone colour, pick a handle) also work and save real accounts. **Almost everything else is a façade.** Likes are forgotten when you close the app. Every video shows the same eight made-up comments. "Submit Video" fakes an upload bar and throws the file away. The Archive, the paywall, folders, Settings and the brand channel pages don't exist. The biggest problem: **the admin panel and the app aren't connected.** Videos you add in admin go into the database, but the app never reads the database. It plays a list that is typed directly into the code. Any rebuild should start by fixing that.

---

## 1. Which repo is which

You have four GitHub repos with "magic" in the name. Only one of them is the app.

| Repo | What it actually is | Verdict |
|---|---|---|
| **magic** | Empty. No files at all (this report is its first file). | Use it for the rebuild, or delete it. |
| **magic-pwa** | **The real app.** Everything below is about this one. 79 commits, 31 Jul to 12 Oct 2026. | This is what you're rebuilding. |
| **magic-archive** | Design files only: the v9 handoff documents, design tokens, and 34 screen mockups. No app code. It's an older copy of the `design/` folder inside magic-pwa. | Reference material. Keep magic-pwa's copy, which is newer. |
| **magic-tv** | An earlier prototype from June (simpler feed, same videos). It also holds a separate mini-project, "Regift — Magic Earth", a page that logs QR-code scans from 500 shirts and plots them on a globe. It also contains junk: trial font files, build leftovers, a file called `Untitled`. | Retired. Nothing in it needs carrying forward except, perhaps, the shirt-scan idea. |

---

## 2. What it's built with

Plain-English translations in brackets.

- **Next.js 16 + React 19** (the website framework; very current and very standard. Any web developer can pick it up.)
- **A PWA, not a native app** (it runs in Safari and can be "Added to Home Screen", where it feels app-like: full screen, black launch splash, works partly offline. It is not in the App Store.)
- **Mux** (the video hosting service. Every video is a Mux "playback ID". Mux handles streaming quality. This is a good choice and should stay.)
- **Supabase** (the database plus login system. It holds accounts, profiles, and *some* channel/video data; see §4.)
- **Google sign-in** via Supabase. **SMS sign-in** via Twilio is half-wired (see §3).
- **Vercel** (hosting; the live URL is `magic-pwa.vercel.app`).
- **Monument Grotesk** (the licensed brand font).

**Health check I ran:** the app **builds successfully** for production, and the type-checker (an automatic error scan) **passes cleanly**. The code-style checker reports **11 errors and 8 warnings**. They're mostly about React best practice, not crashes. There are **zero automated tests**.

---

## 3. Feature status

### ✅ Works

| Feature | Notes |
|---|---|
| **Video feed: swiping** | Swipe up/down for the next or previous video in a channel. Swipe left/right to change channel. It feels smooth and native, which is where most of the engineering effort went. |
| **Video playback quality** | Preloads neighbouring videos so swipes start instantly. Poster images cover the black flash before a video starts. A lot of iPhone-specific tuning has gone in. |
| **Launch intro** | MAGIC logo on black, fades into the first video, then channel names fade in and out. |
| **Tap / double-tap** | Tap shows the details layer (title, channel, date, "submitted by"). Double-tap switches between full-screen crop and the clip's original shape. Matches the v9 spec. |
| **Sound + screen-awake** | Sound turns on at first touch, there's a mute control, and the screen won't sleep while watching. |
| **Home Screen install** | Proper app icon, black splash screens for every iPhone size, offline fallback. |
| **Google sign-in** | Real, works end to end. |
| **Onboarding front door** | Signed-out users are sent here: "Watch better." → Google → pick phone colour (with live selfie-camera reflection in the phone glass) → pick @handle (live "available/taken" check) → 5-card tour → welcome → zoom-into-feed animation. Saves a real profile to the database. |
| **Deep links (partial)** | A link with `?v=<video>&t=42` opens that clip at 42 seconds, even after going through sign-in. *But the Share button doesn't generate these links; see below.* |
| **Admin page** (`/admin`) | Add a video (by Mux ID), publish/unpublish videos, approve/reject submissions. Protected by an email allowlist or a temporary passcode. *But see §4: what you do here doesn't reach the app.* |

### 🟡 Half-finished (looks real, isn't)

| Feature | What's actually happening |
|---|---|
| **Like** | The heart toggles, but it's stored only in the browser tab's memory. **Close the app and every like is gone.** Nothing is saved to the database, and nobody else's likes are counted. |
| **Comments** | **The same 8 fake comments ("Sander Iyer", "Li Wei"…) appear on every single video**, under a hard-coded "39 Comments" header. The "Add a comment" box is a button that does nothing. |
| **Share** | Copy Link, Messages, WhatsApp, Mail and X all work, **but they always share `magic-pwa.vercel.app/feed`**, the generic front page, never the clip you're watching. The deep-link system exists; Share just doesn't use it. |
| **Submit Video** | You pick a file, a progress bar animates for 2 seconds, and it says done. **The file is never uploaded anywhere.** The code itself labels this "MOCK upload". |
| **"Tools" sheet** | Buttons for "Save to Archive", "License for Commercial Use" and "Creator Profile". **None of them do anything when tapped.** |
| **"More" sheet** | Sign in/out works. "Contributors", "Information", "Copyright" and "Guidelines" do nothing. |
| **Apple sign-in** | Button shows "Apple sign-in coming soon." (The spec says this is intentional for now.) |
| **Phone/SMS sign-in** | Actually wired to Twilio, but **only inside an older sign-in pop-up in the feed**. The new onboarding screen says "coming soon". So there are two sign-in UIs, and they don't agree. |
| **Returning users** | If an existing user signs in on a new phone, they're walked through the whole onboarding again: colour, handle, tour. Their handle field is pre-filled from their Google name, not their existing handle, and saving **overwrites their existing profile**. The spec's "Welcome back" screen isn't built. |
| **Branded website channels** | TelfarTV, MythMag, EdgLrd, DisMag, UbuWeb, Walker and PS1 (YouTube) are in the code but **switched off** (`demoReady: false`). When switched on, they show the partner's website in a frame, but the page is made un-clickable, so the spec's "cut a hole so their links work" isn't done. |
| **Visual design vs. v9 spec** | The feed still uses the **older design**: a footer with Tools / Like / Comment / Share pills and a "More" sheet. The v9 handoff's right-hand rail, its blue ■ Quick Archive button and its redesigned Comments/Share screens haven't been built. The animated blue circle-and-square loader isn't built either. |

### ❌ Not built at all

Everything below is fully designed in the v9 handoff (`design/HANDOFF.md`, `design/ARCHIVE.md`) but has **no code**:

- **The Archive** (`archive.magicearth.tv`): browse, search, filter, saved clips, folders, asset pages. **This is the business model** (§0a of the handoff: "The Archive is the paywall").
- **Payments, the 7-day trial, subscriptions, the trial-offer sheet.** No payment provider is connected. There are no entitlement records.
- **Saving clips**, **folders**, **AI mood tags** ("Sorted by MAGIC").
- **■ Quick Archive** panel.
- **Settings** page.
- **Brand channel home** page (logo, Follow, schedule).
- **Profile / submitter** pages, **Follow**.
- **Notifications.**
- **Desktop version** (the spec says it comes second).
- **Analytics.** A `clicks` table exists in the database, but nothing ever writes to it.

---

## 4. How the video feed and channels are structured

### The mental model

The feed is a **grid**:

```
            ← swipe →  changes CHANNEL
   ┌─────────┬─────────┬─────────┬───────────┬─────────┬─────────┐
   │ YOUTH   │ FIGHT   │ SOUND   │ LOT RADIO │ VISION  │ TECH    │
   ├─────────┼─────────┼─────────┼───────────┼─────────┼─────────┤
 ↑ │ video 1 │ video 1 │ video 1 │ video 1   │ video 1 │ video 1 │
 │ │ video 2 │ video 2 │ video 2 │           │ video 2 │ video 2 │
 ↓ │ video 3 │ video 3 │ video 3 │           │ video 3 │ video 3 │
   └─────────┴─────────┴─────────┴───────────┴─────────┴─────────┘
   swipe ↑↓ changes VIDEO within the channel (loops)
```

- **6 channels are live**, with **16 videos** in total (3 each, except Lot Radio with 1). Channels loop: swipe past Tech and you're back at Youth.
- **7 more channels exist but are hidden**. These are the partner-website and YouTube channels listed above.
- Each **channel** has a name, an "ambassador" (the partner or curator), and a list of videos.
- Each **video** has a Mux ID, title, date, time, "submitted by", "via", description and position.
- The app remembers which channel and video you were on, but only until the app is fully closed.

Behind the scenes, the app keeps **4 videos warm at once**: the current one, the next one down, and the neighbours left and right. That's why swipes feel instant. It's also why this code is so intricate.

### ⚠️ The critical structural problem: there are two sources of truth, and the app ignores the database

There are **two separate copies** of the channel and video list:

1. **A list typed directly into the code** (`lib/channels.ts`). **This is what users see.**
2. **The Supabase database** (`channels` and `videos` tables). **This is what the admin page edits.**

The developer built a bridge between them (`getChannels`, which reads the database and falls back to the code list), **but never plugged it in**. Nothing calls it. So:

- **Adding a video in `/admin` does nothing visible.** It goes into the database, but the app never looks there.
- **Unpublishing a video in `/admin` does nothing.** It keeps playing.
- **Changing what's on air requires a developer to edit code and redeploy.**
- The two copies **have already drifted**: Lot Radio exists in the code but not in the database.

Similarly, **approving a submission in admin just marks it "approved"**. It doesn't turn it into a video. And since Submit Video never uploads anything (§3), there are no real submissions to approve anyway.

### Other database issues worth knowing

- **Two competing "user" tables.** `profiles` (handle, phone colour) is filled when someone signs up. `users` (older, holds the admin role) is never filled for normal sign-ups. The likes and comments tables are linked to `users`. So **when someone finally wires up likes and comments, they'll fail for every normal user** until this is untangled.
- **The video date and time are stored as free text** ("March 14, 2025", "8.22pm") rather than real dates. That means no sorting by date, no "this week", no time zones.
- The video tables don't match the Archive spec's data model (clip IDs like `MA-004217`, mood tags, aspect ratio, resolution, and so on). They will need redesigning anyway.

---

## 5. Code quality, frankly

- **One 5,000-line file runs the entire feed** (`app/feed/page.tsx`): swiping, the intro, all the pop-up sheets, sign-in, submit, comments, share, the debug panel. It's like a building where every room shares one electrical circuit: any change risks breaking something unrelated. This is the main reason progress has slowed into long stretches of "fix the flash on swipe-back" commits.
- **The code is heavily commented, which is genuinely helpful.** A new developer can understand *why* things are the way they are, especially the iPhone video workarounds. That knowledge is valuable; don't throw it away in a rebuild.
- **Lots of hard-coded values** that should be settings or data: the share URL, "39 Comments", the fake comments, and the developer's personal network addresses in the config.
- **No automated tests.** Every change is verified by hand on a phone.
- **The README is the unedited default template.** There are no setup instructions for a new developer.
- **The admin "passcode" login** is labelled temporary in the code and should be removed once email login for admins is trusted.
- **Security is mostly sensible:** database permissions are locked down, the profile table is protected, and the sign-in redirect is guarded. The magic-tv shirt-scan function sends visitor IP addresses to a third-party location service over an unencrypted connection. That's a privacy issue if it's still live.

---

## 6. What I'd fix first in a rebuild

In order. Each step unblocks the next.

### 1. Make the database the only source of truth for channels and videos
Delete the hard-coded list. Have the app load channels and videos from Supabase. Then the admin panel actually controls what's on air, you can add content without a developer, and everything after this has a real foundation. **This is small in effort and the biggest unlock.**

### 2. Fix the user model before adding any social features
Merge the two user tables into one `profiles` table with a role (viewer / curator / admin). Re-link likes and comments to it. Make returning users skip onboarding and keep their handle. If you skip this, step 4 breaks.

### 3. Break up the 5,000-line feed file, *keeping* the video engine
The swipe and preload engine is the hardest-won part of the codebase. Lift it out intact into its own piece. Then rebuild each overlay (comments, share, the details layer and rail) as its own separate part, built to the v9 design rather than the old one.

### 4. Make the "fake" features real, cheapest first
- **Share** the actual clip link (`?v=…&t=…`); this is nearly free, since the plumbing exists.
- **Likes** saved to the database, so they persist and can be counted.
- **Comments** saved and loaded for real, with sign-in required to post.
- **Submit Video** uploading to Mux and creating a submission, with "Approve" in admin turning it into a published video.
- **Pick one sign-in screen**, delete the other, and decide whether SMS sign-in stays.

### 5. Build the thing that makes money: Save → trial → Archive
Until this exists, MAGIC has no revenue path. Order: a payment provider (Stripe is suggested in the spec) and an "entitlement" record per account → Save button + trial-offer sheet → Quick Archive → full Archive site. The spec's data model in `ARCHIVE.md` §5 is a good starting point; build the new database to it from day one.

### 6. Then the long tail
Settings, brand channel pages, profiles and Follow, AI mood tagging, notifications, the website-channel "cutout", desktop.

### Alongside all of it
- Add a basic set of automated checks for the swipe feed so future changes can't silently break it.
- Write a real README so a new developer can run it in under an hour.
- Archive `magic-tv` and `magic-archive` (or fold the design files into one place) so nobody builds against the wrong repo.

---

## 7. Decisions you'll need to make (from the spec's own "Open" list)

These block the money features in step 5:

1. After a trial lapses: keep the user's saves locked, or delete them?
2. Does the trial need a card up front?
3. Pricing and plan names for small creatives; payment provider.
4. Brand deals: seats per organisation, and do brands get curator tools?
5. Are Apple sign-in and SMS sign-in needed at launch, or is Google alone enough?
