# CareThread

made by Salem Surur for the 2026 Congressional App Challenge (district CA36, Santa Monica)

## why i made this

I have relatives with autism and my grandmother has dementia. Whenever different people take care of them, like family, friends or a helper, a lot gets lost between shifts. Someone forgets to say that the evening medicine was already given, or that a loud noise upset them earlier. I wanted one place where every caregiver can see what happened and what to watch out for, so the next person doesn't start from zero.

## what it does

- **Today** shows the latest handoff note, a quick summary of today's check-ins, the medication list and a care timeline.
- **Handoff** lets a caregiver start and end a shift and leave a note for the next person.
- **Log** is the form for a check-in. You put in mood, medicine, routine, behaviors, triggers and a note.
- **Patterns** finds things in the entries, like which triggers show up the most and how mood changes over the weeks. It also makes a report you can bring to a doctor.
- **Profile** and **Household** keep the person's info (triggers, what calms them, medications) and who is helping.

Everything is in one file, `index.html`, and it works in any browser even offline. Your data is saved in your own browser (localStorage). That means each device saves its own entries and they do NOT sync between devices yet. That is the first thing i want to fix in a 2.0.

## how this was built (the honest part)

The first version of the app was made by me. I designed the screens and wrote the base code myself. That includes the log form, the medication tracker with sign-offs, shifts and handoff notes, the care profile, the household page and the Patterns screen.

Near the end of the project i used AI (Claude) as a coding helper. It helped me with
- polishing how the app looks and fixing bugs
- the privacy step that swaps names for codes before anything is sent
- the doctor report
- the numbers and charts on the Patterns screen
- the automated tests
- the example people and their data
- the logo

I decided what to change, looked at every change, ran the app and the tests, and chose what to keep.

Claude is also part of the app itself. It writes the "Today's entries at a glance" summary and helps write the doctor report. Those parts are optional. If the AI isn't working, the app shows a normal version it builds on its own.

All the people and notes in the app are made up. No real person's info is in here.

## the AI parts

There is one server function, `/api/care-insights` (on Vercel). It asks Claude to write short plain text. Each place first shows a built in version, and only changes after something is added, edited or removed. If the server is missing, offline, busy or out of quota, the built in version stays and a small notice says why.

- **Today's entries at a glance** (Today and Handoff) gets rewritten after every entry change. It can also show up to three "things to be mindful of" reminders from the profile and the last 7 days. It uses the fast model.
- **Doctor report** (Patterns) is first written on the device from the records. After a change the AI rewrites the summary, important events, patterns and questions. If a caregiver edits the text, it never gets overwritten by itself. The refresh button does replace it.
- **AI reflection** (Patterns) only runs when you press its button.

## files

```
index.html              the whole app (screens, styles and logic)
api/care-insights.js    the only server code. checks the request, limits requests, calls Claude, checks the reply
vercel.json             function time limit and security headers
package.json            just the node version, no dependencies
.env.example            names of the environment variables (no real values)
test/                   api.test.js, e2e.js, dev-server.js (a local server with a fake Claude, never deployed)
logo.svg                the logo (one thread that makes a heart, with an orange dot for the handoff)
```

## shifts

"Start my shift" saves who is on shift, for which person and when it started. You don't need an entry first. It stays after you refresh and there is only ever one active shift per person. "End shift" saves the end time, shows how long it was and offers a handoff note. Then it moves to the shift history. Entries made by the person on shift get tagged with that shift. Shifts are also saved only in that browser.

## how to run it

Local (no key needed for the tests)
- `npm run test:api` checks the server code with a fake Claude
- `npm run test:e2e` runs browser tests (needs Playwright)
- `node test/dev-server.js` runs the app on port 3111 with a fake Claude, its text starts with `[fake AI]`

With the real AI, put the key in `.env.local` and use `vercel dev`.

## deploying to Vercel

1. Put the project on GitHub. Check that no `.env` file is in it.
2. Make an API key in the Anthropic Console and set a low monthly spend limit. This is the real protection for the cost.
3. In Vercel, add a new project from the repo. Framework preset is Other, leave the build command and output folder empty.
4. In Settings > Environment Variables add `ANTHROPIC_API_KEY` (production). Also `ANTHROPIC_MODEL` (like `claude-sonnet-4-6`) and optionally `ANTHROPIC_FAST_MODEL` (a Haiku model) for the quick glance card.
5. Deploy. A new variable only works for deploys made after you save it.
6. Test it with `curl -i -X POST https://YOUR-SITE.vercel.app/api/care-insights -H "Content-Type: application/json" -d "{}"`. A `400` means the function works and the key is set. `503 not_configured` means the key is missing. `404` means the function didn't deploy.
7. Optional but good, add a Vercel firewall rate limit on `/api/care-insights` (30 requests per 10 minutes per IP).

## privacy, what leaves the device

Only when the AI runs. That is after an entry changes (glance), after a change on the Patterns screen (doctor report) or when the reflection button is pressed.

Sent is the age group, the calculated numbers, the entries and their notes (long notes are cut down to the most important sentences), recent handoff notes and care log details, and the profile's triggers and comfort preferences. The person's name and the medication, condition and doctor names from the profile are swapped for placeholders like `[NAME]` and `[MED1]`, and the real words are put back on the screen only.

Never sent are phone numbers, emergency notes, caregiver names and household details.

Swapping names is best effort. If someone misspells a name, or types a drug that isn't in the profile, or another person's name in a note, it gets sent as written. So only use made up people for a public demo. For real families, get their ok first.

## limits of the request limiter

The app checks that the request comes from the same site, only takes JSON, caps the body at 12 KB and caps how much comes back. It allows 30 AI requests per visitor per 10 minutes and 300 per hour per server.

This limiter lives in memory, so it is NOT production grade. It resets when the server restarts, every instance has its own count, and someone changing IPs gets a fresh allowance. The spend limit in Anthropic and the Vercel firewall rule are what really cap the cost. A real version would need a shared store like Redis and a login.

## safety rules for the AI

The instructions tell Claude to only repeat what caregivers reported, not diagnose anything, not give medication or treatment advice, not give doses, not say one thing caused another, say when there isn't enough data, and ignore any instructions that are hidden inside the data. A small filter also drops any line that gives medication advice or names a diagnosis, and tells the caregiver when it did.

The filter is just a backup and not a promise, so read real outputs before showing it to people.

## what i'd do next (2.0)

- sync between devices so every caregiver sees the same log (right now each device is separate)
- real logins for each caregiver
- a proper shared rate limit
- more languages
