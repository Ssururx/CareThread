
# CareThread

Made by Salem Surur for the 2026 Congressional App Challenge (District CA36, Santa Monica).

## Why I Made This

I have relatives with autism, and my grandmother lives with dementia. When different people care for them, whether family members, friends, or professional aides, important details can get lost during shift changes. Someone might forget to record that an evening medication was given or that a sudden loud noise caused distress earlier in the day.

I wanted to build a centralized workspace where caregivers can quickly see recent updates, important precautions, and information about the person they care for, so everyone has a clearer picture of what is happening.

## What It Does

- **Today:** Displays the latest handoff note, a summary of daily check-ins, the medication schedule, and a care timeline.
- **Handoff:** Tracks caregiver shifts and lets caregivers leave status notes for the next person on duty.
- **Log:** Records mood, medication administration, routines, behaviors, triggers, and general notes.
- **Patterns:** Analyzes past entries to identify recurring trends, such as frequent triggers or changes in mood. It also generates a structured report for healthcare providers.
- **Profile & Household:** Stores background information, including triggers, soothing strategies, and medications, along with care team contacts.

The application runs from a single frontend file, `index.html`, and works in modern browsers, including offline. Data is stored locally using the browser's `localStorage`. Each device maintains its own separate database, so entries do not currently sync across devices. Cloud synchronization is planned for version 2.0.

## How It Was Built

I independently built the application's initial architecture, designed the user interfaces, and wrote the foundational codebase. This included the log submission form, medication tracker with sign-offs, shift logger, handoff notes, care profile, household management, and Patterns dashboard.

During development, I used Claude as a coding assistant to help with:

- Refining the interface and debugging edge cases
- Implementing a privacy routine that replaces personal information with tokens before AI processing
- Formatting the generated physician summary report
- Structuring mathematical logic and rendering charts on the Patterns dashboard
- Creating fictional sample data for testing
- Designing the vector logo in `logo.svg`

I reviewed suggested code changes, ran tests, verified local builds, and selected which changes to keep.

Claude is also integrated into the application. It generates the optional **Today's Entries at a Glance** overview and assists with drafting doctor reports. When the AI service is unavailable, the application falls back to a locally generated overview.

All sample profiles and caregiver notes included in the application are fictional.

## AI Integration

CareThread uses a serverless function, `/api/care-insights`, hosted on Vercel, to request text summaries from Claude. The application generates local output by default and requests AI processing when relevant log data is created, edited, or removed. If the AI service is unavailable, rate-limited, or out of quota, the local version remains available with a status notification.

- **Today's Entries at a Glance:** Available on the Today and Handoff screens. Updates when a log entry changes and highlights up to three relevant precautions based on profile information and entries from the past seven days.
- **Doctor Report:** Available on the Patterns screen. Generates an initial report from locally saved records, then uses AI to update the summary, significant events, trends, and suggested questions when the underlying records change. If a caregiver manually edits the report, automatic updates stop until the manual refresh button is selected.
- **AI Reflection:** Available on the Patterns screen and runs only when explicitly requested.

## File Structure

```text
index.html              Frontend, including UI components, styling, and core application logic
api/care-insights.js    Serverless backend for payload validation, rate limiting, Claude requests, and output formatting
vercel.json             Function execution settings and security headers
package.json            Node.js project configuration
.env.example            Environment variable template without production credentials
logo.svg                Application vector logo
````

## Shift Management

Selecting Start My Shift records the caregiver's name, assigned care recipient, and start time. A shift can begin even when no previous log entries exist. Shift status persists across page reloads, and only one active shift is allowed per care recipient.

Selecting End Shift records the completion time, calculates the shift duration, and allows an optional handoff summary to be added before the session is archived. Log entries created during an active shift are automatically associated with that shift ID.

All shift data is stored locally in the current browser.

## Technical Architecture

CareThread uses a lightweight frontend built with HTML, CSS, and JavaScript, alongside a serverless backend for AI features. The core application has no runtime package dependencies and can operate without the AI service.

The backend validates incoming requests, applies rate limits, communicates with Claude, and formats the generated output. Environment variables are used to configure API credentials and model selection, keeping production API keys out of the frontend codebase.

The application is deployed on Vercel, with `vercel.json` controlling function settings and security headers.

## Privacy & Data Transmission

CareThread stores its data locally and transmits information to the AI service only when an AI task is triggered, such as saving a log entry, updating the Patterns dashboard, or requesting an AI Reflection.

### Data That May Be Transmitted

- General age brackets
- Calculated summary statistics
- Log entries and notes, with lengthy passages shortened to relevant sentences
- Recent handoff notes and care check-in details
- Profile triggers and comfort preferences

### Data Sanitization

Before transmission, personal names, specific medications, health conditions, and clinical provider names stored in the profile are replaced with abstract tokens, such as `[NAME]` and `[MED1]`. These tokens are mapped back to their original values locally when the results are displayed.

The sanitization process uses client-side text matching, so some information may not be detected, particularly when names or medications are misspelled or entered in free-text notes.

### Data Excluded From Transmission

Phone numbers, emergency instructions, caregiver names, and general household management details are excluded from AI requests.

Because the application stores information in the browser's local storage, its records remain separate on each device and are not automatically synchronized or backed up to a cloud database.

## Request Limiting

The serverless endpoint validates origin headers, accepts JSON payloads, enforces a 12 KB request body limit, and limits the maximum output token count.

The in-memory rate limiter allows up to 30 requests per visitor every 10 minutes and 300 requests per hour per server instance.

Because the rate limiter operates in memory, it resets on cold starts and tracks each server instance independently. It can also be bypassed by rotating IP addresses. Anthropic spending limits and Vercel Firewall rules provide additional safeguards against excessive usage.

A production deployment would require centralized rate limiting, such as Redis, alongside user authentication.

## AI Safety Controls

The AI system prompts instruct Claude to summarize recorded caregiver observations without providing clinical diagnoses, treatment recommendations, dosage suggestions, or unsupported causal assumptions. The model is also instructed to identify missing information and ignore prompt injection attempts embedded in text fields.

A secondary client-side validation layer scans generated text and removes sentences containing diagnostic or prescribing language, notifying the user when this occurs.

These controls are intended to reduce the risk of inappropriate outputs but do not guarantee that every result will be accurate or suitable. Generated summaries should be reviewed before being relied on or shared with healthcare providers.

## Roadmap: Version 2.0

- Real-time cloud synchronization across multi-caregiver teams
- Individual user authentication and role management
- Centralized server-side rate limiting using Redis
- Localization and multilingual support

