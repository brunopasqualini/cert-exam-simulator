# Mock Exam Simulator: free AWS practice exams (SAA-C03, DVA-C02)

**Live app: <https://brunopasqualini.github.io/exam-simulator/>** (free, no signup, works offline).

An installable, offline-capable mock exam app. It ships with 1,105 original practice questions for the AWS Certified Solutions Architect Associate (SAA-C03) and Developer Associate (DVA-C02) exams. It runs any certification whose questions follow the JSON format below. Plain HTML, CSS and JavaScript: no build step, no dependencies.

Features: timed and untimed exams, an explanation for every answer, flagging, a score by exam domain, a retest of weak areas, light and dark theme, and a "paste your own JSON" option.

The questions are AI-generated from official documentation and are not official exam content. This is an independent study tool, not affiliated with AWS, Google Cloud, Microsoft or any other provider; the app says so on every screen.

## Structure

```
index.html       the whole app (markup, styles, logic)
manifest.json    PWA manifest
sw.js            service worker: offline cache
banks.json       catalog: certifications, each with a list of question banks
questions/       one folder per provider/exam, one JSON file per bank
scripts/validate-banks.mjs   checks the catalog and every bank
```

## Question banks

A bank is a JSON file:

```json
{
  "examTitle": "AWS Certified Developer - Associate (DVA-C02)",
  "questions": [
    {
      "id": 1,
      "domain": "Domain name from the exam guide",
      "type": "single",
      "question": "Question text",
      "options": ["Option A", "Option B", "Option C", "Option D"],
      "correctAnswers": [1],
      "explanation": "Why the answer is right and why each other option is wrong.",
      "reference": "https://docs.example.com/..."
    }
  ]
}
```

- `type` is `single` (one entry in `correctAnswers`) or `multi` (two or more). `correctAnswers` holds zero-based indexes into `options`.
- `domain` drives the score breakdown. `reference` is an optional `https` link.

Register each file in `banks.json`: a list of certifications, each with an `id`, a `title` and its `banks`. A bank entry needs only `file`, the path from the repo root (`description` is optional):

```json
{ "certifications": [
  { "id": "aws-dva-c02", "title": "AWS Certified Developer - Associate (DVA-C02)",
    "banks": [ { "file": "questions/aws/dva-c02/DVA-C02_01.json" } ] }
] }
```

Check your changes with `node scripts/validate-banks.mjs`. To contribute a bank, see [CONTRIBUTING.md](CONTRIBUTING.md).

How the app uses the catalog:

- Each exam run draws up to 65 random questions from the chosen bank.
- A bank shows in the dropdown as "Question Bank N", N being its position in the certification's `banks` array.
- A bank's id is its file name without folder and `.json` (`DVA-C02_01`). Progress, flags, timers and history in `localStorage` are keyed by it, so **do not rename a bank file** once people use it: their saved progress would be orphaned. Ids only need to be unique within a certification.
- The last selected certification and bank are remembered. If one was removed from the catalog, the app falls back to the first available and shows a notice.
- Progress and history stay in the browser (`localStorage`); nothing syncs or is sent anywhere.

## Install and offline use

Over HTTPS: Chrome and Edge show an **Install app** button, Android Chrome has **Add to Home screen**, iOS Safari has **Share → Add to Home Screen**. After the first visit the app works offline (the shell, plus every bank you have opened).

The service worker caches the shell first, and fetches question JSON stale-while-revalidate, so new or edited banks appear on the next load when online. `CACHE_VERSION` in `sw.js` is the placeholder `__BUILD_SHA__`; the site build replaces it with the commit SHA so every deploy clears old caches. Never edit it by hand.

## Deployment

This repo does not host the app. A push to `main` runs `.github/workflows/ci.yml` (validates the banks), then sends a `repository_dispatch` event (`project-updated`) to [brunopasqualini.github.io](https://github.com/brunopasqualini/brunopasqualini.github.io), which copies the app into `/exam-simulator/`, stamps `sw.js` and deploys the site.

Needs the secret `SITE_DISPATCH_TOKEN` in this repo: a fine-grained token on `brunopasqualini/brunopasqualini.github.io` with **Contents: Read and write** (required to send `repository_dispatch`).

To run locally, serve this folder with any static server, for example `python3 -m http.server`.
