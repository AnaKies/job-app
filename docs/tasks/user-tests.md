# User Tests

See also: [overview.md](./overview.md)

## [F1 Foundation Skeleton](./overview.md)

Go to the app in the browser.
Confirm that the web app opens without a crash and shows the initial shell page.

## [F2 Shared Contracts And User Context](./overview.md)

Open the app and trigger one basic data request.
Confirm that the request works and the app does not show an auth or scope error in normal single-user use.

## [F3 Resume Management](./overview.md)

Go to the resume area.
Upload a resume file and confirm that it appears as a stored resume version in the UI.

## [F4 Company Discovery](./overview.md)

Enter a natural-language search prompt such as a target city and company type.
Confirm that the system returns a visible list of candidate companies.

## [F5 ATS Scraping Pipeline](./overview.md)

Run a discovery flow that leads to known supported career pages.
Confirm that the app can continue from discovered companies toward job data without a visible crash or dead end.

## [F6 Opportunity Feed](./overview.md)

Open the opportunity feed.
Confirm that visible job cards appear with company, role, and basic listing information.

## [F7 Matching And Recommendation](./overview.md)

Open one opportunity with at least one stored resume version.
Confirm that the app shows a visible match score, explanation, and recommended resume guidance.

## [F8 Application Tracking](./overview.md)

Mark one opportunity as applied and then update it to another visible outcome such as interview or rejection.
Confirm that the updated state is shown in the app.

## [F9 Historical Retrieval And Insights](./overview.md)

After recording multiple application outcomes, revisit a new similar opportunity.
Confirm that the app shows insight-informed guidance or learning-based recommendation behavior.

## [F10 Skill-Gap And Resume Guidance](./overview.md)

Open the guidance area for a relevant opportunity.
Confirm that the app shows a visible skill-gap or resume-improvement suggestion.

## [F11 Cover-Letter Scaffolding](./overview.md)

Open the cover-letter support flow for a relevant opportunity.
Confirm that the app shows structured bullet-point scaffolding rather than a blank result.

## [F12 Runtime Hardening And Release Readiness](./overview.md)

Repeat the critical visible flow:
upload resume -> discover companies -> open opportunities -> view match -> mark applied.
Confirm that the flow works without visible blocking errors.
