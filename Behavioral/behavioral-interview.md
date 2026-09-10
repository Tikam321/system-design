# Anaplan Cultural Fit / Behavioral Interview Prep — Tikam Swasi

> Role: Sr. Fullstack Engineer (Java/React) — Operational Data Layer (ODL) Team
> Format: STAR (Situation, Task, Action, Result) where applicable
> Note: Fill in the `[ ]` placeholders with real numbers before your interview — they make your answers significantly stronger.

---

## 1. Tell me about yourself (in 2 sentences)

Hi, I'm Tikam Swasia, a full-stack engineer with 5 years of experience, currently at Samsung working on an admin portal for policy and access management. I enjoy taking end-to-end ownership — writing clean, modular REST APIs in Java and Spring Boot, optimizing their response time, and also building the React frontend for the same product, including improving the overall user experience.

---

## 2. What are three words that other people would use to describe you?

**Responsive, Proactive, Collaborative**

- **Responsive** — when my manager or teammates ping me with a question or blocker, I try to get back quickly so work doesn't stall.
- **Proactive** — I like to actively participate in product discussions and design decisions, not just wait for tickets to be assigned.
- **Collaborative** — I work closely with my team to make sure we're delivering good user experience and hitting deadlines, sometimes ahead of schedule.

---

## 3. What are your three strengths and three weaknesses?

**Strengths:** Responsive, Proactive, Collaborative *(same as above — can reuse or vary phrasing if asked in same interview)*

**Weaknesses:**

1. **Delegation / letting go of ownership**
   One thing I've noticed about myself is that because I like taking end-to-end ownership, I sometimes hesitate to hand off parts of a task to teammates, even when it would be faster to split the work. I've been working on this by being more intentional about breaking tasks down early and assigning clear ownership within the team instead of defaulting to doing it myself.

2. **Saying no / overcommitting**
   I also tend to say yes to additional tasks even when my plate is full, because I don't like blocking others. This has sometimes stretched my bandwidth thin. I've started being more upfront with my manager about my current load before taking on new work, so priorities are set clearly instead of me just absorbing everything.

3. **Estimating timelines**
   Early on, I used to underestimate how long certain tasks would take, especially ones involving unfamiliar parts of the codebase. I've improved this by breaking tasks into smaller sub-tasks before estimating, and building in buffer time for the unexpected.

---

## 4. Tell me about a time you faced a challenge — how did you deal with it?

**Situation:** I was originally a backend developer on my team, but at one point I was asked to also start contributing to the frontend, which used React — a codebase and stack I hadn't worked in before.

**Task:** My job was to start picking up frontend bugs and shipping fixes through PRs, despite having no prior hands-on React experience.

**Action:** I broke my ramp-up into two parallel tracks — learning React fundamentals through tutorials, while simultaneously reading through our actual codebase to see those concepts applied in context. I intentionally started with smaller, low-risk bugs first to build confidence, and gradually moved on to more complex ones as my understanding improved.

**Result:** Within `[X weeks]`, I was able to independently pick up and resolve more advanced frontend bugs, and eventually became comfortable enough to build full features on the frontend — which is part of what I do now on the admin portal project. It taught me that a structured, incremental approach is the fastest way to get productive in an unfamiliar domain.

> ⚠️ **TODO:** Add a real timeframe and, ideally, a specific bug/feature you're proud of from this ramp-up.

---

## 5. What's the biggest challenge you've faced professionally, and how did you resolve it?

**Situation:** We onboarded a new client onto our platform, which meant setting up a new environment for them. This required adding feature-flag-based permission controls across our admin portal, which has around 30 menus — so any client-specific menu/feature could be selectively enabled or restricted.

**Task:** We had only about a week to implement and test these changes across all 30 menus before the client's go-live.

**Action:** Instead of the team trying to touch all menus simultaneously, we split the work by domain — each developer owned a specific set of menus end-to-end, which avoided merge conflicts and let each of us move fast within our own scope. I picked up my assigned domains, implemented the flag-based access logic, and tested each one thoroughly before moving to the next.

**Result:** We delivered all the changes within the week, the stakeholders reviewed and signed off, and we deployed to production with zero post-release bugs. The client environment went live on schedule, and the team was specifically appreciated for handling the tight timeline well. What I took away from this is that under pressure, dividing work clearly by ownership — rather than everyone touching everything — is what actually makes tight deadlines achievable without chaos.

---

## 6. Describe a difficult workplace situation and how you overcame it.

**Situation:** While testing one of our features, I found a bug — when an ID field was empty, it caused a null pointer exception that broke the page.

**Task:** I wanted to fix this properly, not just patch it — so I suggested we build a common null-safety utility component and apply it across all menus in the codebase, to prevent similar issues elsewhere.

**Action:** My teammate disagreed, concerned that rolling out a shared component across the whole codebase at once could introduce regressions or unintended side effects in other areas. Rather than pushing my approach or just going along with theirs, I set up a quick discussion with my teammate, our manager, and the stakeholder to align on the right path.

**Result:** We agreed on a middle ground — fix the immediate breaking bug right away, and plan the common null-safety component as a separate, more controlled rollout across the codebase later. This avoided both the immediate production risk and the risk of a rushed, broad change. It taught me that disagreements are best resolved through open discussion and getting the right people in the room, rather than debating back and forth over chat or trying to convince the other person unilaterally.

---

## 7. Have you faced a challenge where you had to find a resource/help? What did you do and what was the outcome?

**Situation:** I was building an automation tool for my team using Python and LLM integrations — both of which were new to me at the time.

**Task:** I needed to get this tool working quickly since it was meant to reduce manual, repetitive work for the team.

**Action:** Rather than spending days figuring things out alone, I reached out to a colleague who had prior experience building similar automation tools with LLMs. He walked me through key parts of the setup, including configuration and approval processes I wasn't familiar with, which saved me a lot of trial and error.

**Result:** With his guidance, I was able to build and ship the automation tool much faster than I would have alone, and it ended up `[reducing a manual process from X to Y / saving the team roughly X hours per week]`. It reinforced for me that asking for help from someone with the right expertise isn't a shortcut — it's often the fastest and smartest path to a good outcome.

> ⚠️ **TODO:** Add a rough estimate of time/effort saved by the automation tool.

---

## 8. Tell me about an achievement you're proud of.

**Situation:** While testing our frontend portal, I noticed the login/landing page was taking noticeably long to load — this wasn't something I was assigned to fix, I just noticed it as a poor user experience.

**Task:** I decided to take ownership of investigating and fixing it, even though it wasn't formally on my plate.

**Action:** I researched the root cause and found that the entire JavaScript bundle was being loaded upfront on the initial page. I proposed and implemented **code splitting** — breaking the bundle into smaller chunks so the browser only loads what's needed for the initial page, rather than the whole application at once. I discussed the approach with my manager before rolling it out.

**Result:** This reduced the initial page load time significantly `[add rough % or before/after seconds]` and made the app noticeably more responsive. My manager and stakeholders specifically appreciated the improvement. I'm proud of this one because nobody asked me to fix it — I noticed the gap myself, took ownership, and drove it end-to-end.

> ⚠️ **TODO:** Add a before/after load time or % improvement.

---

## 9. Why Anaplan? Why this role specifically?

*(Tailored to the Sr. Fullstack — ODL Team JD)*

What drew me to this role specifically is that it's not just frontend development — it's owning the full data-delivery experience layer, from REST integration patterns down to caching and performance strategy, which is exactly the kind of work I've been doing on my current project. On the admin portal at Samsung, I've worked across the stack — building and optimizing REST APIs in Java/Spring Boot, improving response times through caching, query optimization, and pagination, and also owning the React frontend, including performance work like code splitting to cut down load times. So the caching-and-performance ownership this role asks for isn't new to me — it's the part of engineering I actually enjoy most, because it's where technical decisions translate directly into whether the product feels fast and usable.

I'm also excited about the platform context here — sitting on top of an event-driven data pipeline (Avro, Flink, Databricks) and being responsible for making that data feel fast and trustworthy for BI and product teams downstream. That's a step up in scale and system complexity from what I work with today, and it's exactly the kind of growth I'm looking for — moving from owning features end-to-end to thinking about architecture and performance at a platform level.

Lastly, this role's emphasis on mentoring and driving best practices across a team really appeals to me. `[If true: add a real mentoring example — e.g., helping a teammate ramp up on React, the way you did.]`

> ⚠️ **TODO:** Confirm/add a real mentoring example, or soften this line if you don't have one yet.

---

## General reminders before the interview

- Avoid filler phrases in delivery: "so yeah", "that's all", "so so".
- Every STAR answer lands harder with a **number** — time saved, % improvement, team size, deadline length. Fill in the TODOs above before interview day.
- This role (ODL team) is about the **data delivery/visualization layer** on top of Anaplan's core platform — not Anaplan's core Hyperblock planning engine. Be honest about which JD-listed tools you have experience with (React/TypeScript/Spring Boot — yes) vs. which you don't yet (Kotlin, Flink, Databricks, Kubernetes, observability tools) — and pivot to your proven ability to ramp up quickly (see Q4 and Q7 answers) if asked.
- Anaplan's 8 core values: **Innovative, Accountable, Collaborative, Transparent, Resilient, Empathetic, Authentic, Learner.** Many answers above map naturally to "Learner" (Q4, Q7), "Collaborative" (Q6, Q2), and "Accountable" (Q5, Q8).
