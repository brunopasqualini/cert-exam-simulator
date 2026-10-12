# Contributing a question bank

The simulator reads plain JSON question banks, so you can add practice questions for any exam or provider (AWS, Google Cloud, Azure, Kubernetes, ...) without touching the app code. The format is in the [README](README.md#question-banks).

## Rules

- **Original questions only.** No real exam questions, braindumps, or questions copied or lightly reworded from paid courses or banks.
- **Base them on official documentation.** AI assistance is fine, but check every question against the provider's docs and put the `https` link in `reference`. A wrong "correct answer" is worse than no question.
- **Explain every answer:** why the right option is right and why each wrong option is wrong.
- Banks are presented as practice material: AI-generated or community written, not official, not affiliated with any provider.

## Add a bank

1. Create `questions/<provider>/<exam>/<NAME>_01.json`, for example `questions/gcp/associate-cloud-engineer/ACE_01.json`. About 65 questions suit a timed exam; split larger sets into numbered files.
2. Register it in [`banks.json`](banks.json): add the file to an existing certification's `banks`, or add a certification with its own `id`, `title` and `banks`.
3. Run `node scripts/validate-banks.mjs`. It checks valid JSON, unique ids, answer indexes inside `options`, one answer for `single` and two or more for `multi`, a domain and an explanation on every question, and `https` references.
4. Try it in the app (serve the folder with any static server), then open a pull request. CI runs the same validator.

Fixing a wrong answer, improving an explanation or adding a missing reference is welcome too.

There is no license file yet; ask the maintainer how contributions are licensed if that matters to you.
