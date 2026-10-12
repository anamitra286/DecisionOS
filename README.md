## Abstract

DecisionOS is a personal decision-making web application that helps a user turn an uncertain choice into a clear next step and then learns from how that choice turned out. The user describes a decision, lists the realistic options, and the system presents its reasoning from six perspectives: Context, Preference, Trade-off, Regret, Devil's Advocate, and Decision. It returns a recommendation with a clear next move, and the user can accept or decline it and later log the outcome as good, neutral, or bad. Over time, the logged history supports a regret tracker, a view of personal decision patterns, and a set of low-risk autonomous suggestions that always require the user's confirmation. The project is built as a pnpm monorepo using React, Vite, and Tailwind on the front end, Express 5 on the back end, PostgreSQL with Drizzle ORM for persistence, and Clerk for authentication. The current version is a frontend-first prototype: the decision lifecycle works end to end in the browser, while the server-side store and the intelligence behind the recommendations are still planned.

## Problem Statement

People often face decisions that are important but noisy. The facts are incomplete, feelings pull in different directions, and the same question gets circled for days without resolution. Most tools either give a single answer with no reasoning or present a long list of pros and cons that adds to the noise. Few people go back afterwards to check whether their choice worked out, so they rarely learn how they decide. The result is repeated indecision, regret that is never examined, and no record of what actually helped. DecisionOS addresses this gap by offering a structured, explainable way to reason through a decision and a feedback loop that makes future decisions better informed.

## Objectives

The project sets out to give users a private, calm space to write down a decision and its context in plain language. It aims to present the reasoning in a form that can be inspected, so that the recommendation does not feel like a black box. It supports the full decision lifecycle, from capture and recommendation through acceptance and outcome logging. It also aims to make reflection useful by tracking regret and surfacing patterns in how the user chooses. Finally, it aims to offer proactive help that stays safe, meaning it only suggests reversible, low-stakes actions and never acts without explicit confirmation.

## Proposed System

The proposed system is organized around a single workflow. The user starts a new decision by entering a title, the context that makes it hard, and two options. The system then produces a synthesis in six named perspectives, each with a short summary and an explanation. The Context perspective describes the real shape of the choice. Preference reflects what the user keeps returning to. Trade-off lays out what each path costs. Regret imagines how the user might feel looking back. Devil's Advocate raises the most useful objection. Decision gives a clear next move, often with a condition attached. The user then selects one option, and the system asks whether the recommendation earned their trust. Once the outcome is known, the user logs it. The application keeps a trail of past decisions and uses it to build a regret view, a patterns view, and a set of suggestions. Each suggestion is presented as a private note, and a confirmation dialog makes clear that nothing outside the application is changed.

The design also reflects a deliberate stance on tone and trust. The visual language uses warm parchment, indigo, apricot, and mint tones, and the copy is written to encourage reflection rather than to judge the user's choices.

## System Architecture

The project is a monorepo managed by pnpm workspaces, with the main packages separated into artifacts (the deployable applications) and libraries (shared code). The structure is as follows.

```
DecisionOS (pnpm monorepo, Node.js 24, TypeScript 5.9)
│
├── artifacts/decision-os        React + Vite + Tailwind + shadcn/ui
│     ├── App.tsx                 Shell, routing (wouter), decision state,
│     │                           six-perspective synthesis, history, regret,
│     │                           patterns, suggestions
│     ├── Clerk sign-in / sign-up  Gates the private workspace
│     └── localStorage             Current persistence of decision records
│
├── artifacts/api-server         Express 5 API (port 5000)
│     ├── routes/health.ts         GET /healthz (only live endpoint)
│     └── middlewares/clerkProxyMiddleware.ts
│
├── artifacts/mockup-sandbox     Component preview sandbox (Vite)
│
└── lib/
      ├── db                     PostgreSQL + Drizzle ORM
      ├── api-spec               OpenAPI 3.1 spec (currently health only)
      ├── api-zod                Zod schemas generated with Orval
      └── api-client-react       Generated React Query hooks
```

The front end is a single-page application. It keeps the decision records in a list, with the active decision always in the first position. Records are saved to browser localStorage under a single key whenever they change. Routing is handled by wouter, and authentication by Clerk, which determines whether the user sees a landing page or the private workspace. The back end currently exposes only a health check, but the surrounding tooling (OpenAPI, Orval codegen, Zod validation, and Drizzle with a PostgreSQL connection string) is already in place for a future user-scoped decision store.

## Explanation

The strongest part of the implementation is the user experience of the decision lifecycle. The workspace presents the active decision, the context that matters, the six perspectives, and a recommendation with two buttons the user can tap to choose an option. The follow-up questions about trust and outcome are placed so that the user can close the loop at their own pace. The history view lists past decisions and lets the user return to any of them. The regret and pattern views turn logged outcomes into simple summaries, such as how many choices felt good in hindsight and how often the user acted on a recommendation they accepted.

The code is organized so that the user-facing reasoning is explicit. Each perspective is a named object with its own icon, colour, summary, and explanation, which makes the reasoning easy to extend. The safety boundary is also built into the interface, with the suggestions page stating that only reversible, low-stakes actions appear and that consequential matters always pause for the user.

## Limitations

The most important limitation is that the intelligence described in the product is not yet computed. The six perspectives, the recommendation, the clarity score, and the patterns are presented from fixed content in the source code rather than generated from the user's input. A new decision always receives the first option as its recommendation, regardless of what the user wrote. The clarity signal (82 out of 100) and the values bars (Freedom, Learning, Stability) are constants. The regret chart uses fixed month labels and bar heights, and the header date is hard-coded. The sidebar shows a placeholder user name rather than the signed-in account.

Several interactive elements are placeholders. The search bar and settings button do not function, the notifications button only shows a message, and the "Write a reflection" feature uses a browser alert. The "Save it" confirmation for a suggestion displays a success message but does not store the note anywhere. The check-in reminder is described in the interface but no scheduling exists.

Persistence is also limited. Decisions live only in the browser's localStorage, so they are lost if the user clears their browser data, and they cannot be accessed from another device. The database schema and the API are empty apart from the health route, which means the Clerk-authenticated user does not yet have a server-side record. Finally, there are no automated tests in the repository that I could find, and the project's history consists of only three commits, so its quality assurance is still early.

## Future Scope

The natural next step is to connect the recommendation engine to a real model. The six perspectives can be generated from the user's context and options, with the recommendation and its reasoning produced by a language model and returned as structured data. This would make the synthesis genuinely personal and would allow the trade-offs and devil's advocate points to respond to the actual decision. Recommendations could also be checked against the user's past outcomes, so that the system learns which kinds of advice have worked for them.

On the persistence side, the planned server-backed store should be implemented by defining the decision and outcome tables in Drizzle, exposing them through endpoints described in the OpenAPI spec, and scoping every record to the authenticated Clerk user. This would allow multi-device use and safe backups. The patterns and regret views could then be computed from real data rather than fixed text, and the clarity score could be derived from how complete the user's context is.

Other directions include scheduled check-ins that remind the user to log an outcome, export of the decision journal, and tests for the decision flow and the API. The autonomous suggestions could eventually connect to a calendar or task list, but only after the confirmation step has been proven in use and any action has been made reversible.

## Conclusion

DecisionOS presents a thoughtful approach to personal decision-making. It separates reasoning into six explainable perspectives, gives the user a clear next step, and closes the loop by asking how a choice turned out. The front end delivers a polished and coherent experience for the full lifecycle, and the surrounding monorepo is set up for growth into a real backend. The project is currently a prototype whose recommendations, scores, and patterns are fixed content rather than computed insights, and whose data lives only in the browser. Its value lies in the design of the workflow and the clarity of its intent. Connecting the synthesis to a genuine reasoning engine and moving persistence to the server are the steps that would turn it into a working product.
