# ARK

ARK is a web application for churches and the district offices that oversee them. It handles the parts of a church family's week that involve children and money: checking kids in and out, registering for events and camps, paying, giving, staffing, and sending messages. A ministry program, such as a boys' or girls' program, runs inside the same app as configuration, not as a separate product. It was formerly developed under the name Gather.

This repository is a showcase. It holds documentation only. There is no source code here, and all rights are reserved. The source lives in a private repository.

## Who it serves

- Parents, on a phone: register a child, sign waivers, pay, receive a pass for Sunday drop-off, see messages.
- Volunteers and leaders, at a check-in table: look up a child, check them in and out, see allergy flags, run a roster.
- Church offices and pastors: manage people, events, payments and giving for one church.
- A district office: see every church it oversees, provision a new one, and run schema changes across all of them.

## Systems and their status

BUILT means the code, the pages and tests exist in the source. PARTIAL means part of it exists. Nothing here was run on the day this page was written; the statuses come from reading the source and counting files.

| System | Status | What it does |
|---|---|---|
| Child check-in, events | BUILT | Gate check-in and a muster view for camps and weekends. Pickup uses codes. |
| Child check-in, Sunday kids church | BUILT | The parent's phone shows a QR pass. Staff scan it at drop-off and pick-up. |
| Offline check-in bundle | PARTIAL | A design stub. It is not shipped and not claimed to work. |
| Family profiles and approved pickup adults | BUILT | Households, co-parent invitations, a list of adults approved for pickup, allergy and medical notes per child. |
| Event registration | BUILT | Sessions, pricing, waivers, registration windows, waitlists, invitations, a live event-day view. |
| Background check status tracking | PARTIAL | Records status, expiry and who verified it. It stores no documents, by design. It does not run checks. |
| Role-based access | BUILT | One permission table decides every access check. A snapshot test fails if the table changes unnoticed. |
| Giving and payments | BUILT, switched on by configuration | Event payments, church giving with funds and receipts, and refunds through a hosted payment page. Nothing is charged unless payment keys are configured. |
| Messaging | BUILT for email, PARTIAL for text | In-app messages with email notices through a configurable provider. Text messages use an adapter for a self-run Android gateway. It is off unless configured and falls back to email. |
| Volunteer staffing | PARTIAL | Event staffing roles, schedule stations and rotation groups. A recurring weekly volunteer rota was not found. |
| District control plane | BUILT | A control database tracks districts, schema versions and approvals. Provisioning, migration and tenant export are command line operations. |

## Numbers

Counted on 2026-11-24 from the private repository, last commit dated 2026-10-31.

- 4,606 commits across all branches, of which 2,701 are on the main line. The first commit is dated 2026-09-01.
- 777 test files (files ending in .test.ts). The suite was not run for this page, so no pass count is claimed.
- 122 schema migration versions.
- 474 route registrations in the router, 411 behind a session and 63 public. Counted by text, so a route registered dynamically would not appear.
- 0 runtime npm dependencies. The two development dependencies are the TypeScript compiler and Node type definitions. Two small third-party libraries for QR codes are vendored as source files, as the architecture document explains.
- About 176,000 lines under src and about 185,000 lines under tests.

## How it is built

A single TypeScript server on Node, server-rendered pages, no client framework, no build step, and one SQLite database file per district. [ARCHITECTURE.md](ARCHITECTURE.md) describes the request path, the control plane and the migrations.

The code was written and checked by AI agents directed by one founder, who made every product decision and approved every release. [HOW-IT-IS-BUILT.md](HOW-IT-IS-BUILT.md) describes that method, its scale, and its limits. The product itself makes no calls to any AI service.

## Read next

- [ARCHITECTURE.md](ARCHITECTURE.md): request path, data layout, control plane, adapters.
- [SECURITY.md](SECURITY.md): how children's data is protected, with the gaps listed.
- [HOW-IT-IS-BUILT.md](HOW-IT-IS-BUILT.md): how one founder built it by directing AI agents.

## Not covered here

Which server runs the product, customer counts and pricing are not described in this repository. No outside penetration test is claimed.
