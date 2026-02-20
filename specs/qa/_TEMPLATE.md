# QA Handoff — Batch N: [Title]

## What Changed
[1-2 sentence summary of what was deployed]

## Pages to Visually Inspect
- https://staging.example.com/page1/ — [what to look for on this page]
- https://staging.example.com/page2/ — [what to look for on this page]

## What to Look For
- [Specific visual check #1 — e.g., "Login form should match newsletter form styling"]
- [Specific visual check #2 — e.g., "Error message should be red, not default text color"]
- [Specific visual check #3 — e.g., "Protected content should show sign-in link for logged-out users"]

## Known Risks
- [Areas where the coding agent is least confident]
- [Edge cases that were hard to test locally]

## Verification Commands Already Run
```
$ php -l inc/auth.php
No syntax errors detected

$ curl -s 'https://staging.example.com/' | grep -c 'auth-form'
1

$ git diff --name-only
inc/auth.php (new)
inc/template-tags.php

Pipeline: PASS (run #12345)
```
