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
# Behavioral Interview Answers: Questions 10–20

## 10. Tell me about a time you had to be transparent about a mistake or setback.

### Situation
I was developing a feature that required integrating several APIs and implementing CRUD operations. During development, I missed connecting the delete API on one of the pages. Because we were working toward an early release, I did not test the feature thoroughly enough, and the changes were eventually deployed to production.

### Task
I was responsible for ensuring that the feature and its critical user workflows worked correctly before release.

### Action
After deployment, users reported that they were unable to delete certain entities. I immediately discussed the problem with my manager and the relevant stakeholders, took responsibility for the issue, and explained that insufficient testing had allowed the bug to reach production. I helped identify the root cause, connected the missing API, tested the fix, and supported the redeployment.

### Result
The delete functionality was restored, and users were able to complete the affected operation again. I learned that critical workflows require regression and end-to-end testing, even when there is pressure to release early.

## 11. Tell me about a time you had to learn something quickly to keep up with a project.

### Situation
I was working primarily on a backend development project, but the project requirements also required me to fix several frontend bugs. At that time, I had limited experience with frontend development, so understanding the codebase and fixing issues initially was challenging.

### Task
I needed to learn enough frontend development to resolve the bugs and contribute effectively to the project.

### Action
I started by learning the frontend fundamentals and understanding how the existing codebase was structured. I began with smaller, lower-risk bugs so I could build confidence and learn the application flow. As my understanding improved, I gradually took on more complex frontend issues.

### Result
By the end of the project, I was able to resolve the required frontend bugs, including production issues, and contribute more independently to the frontend work. This taught me that I can adapt quickly to unfamiliar technologies by learning the fundamentals first and progressively taking on more challenging work.

## 12. Tell me about a time you collaborated with a team outside your usual function.

### Situation
I was responsible for implementing access control for an application. It was a complex feature, and the requirements were still evolving because we had a short timeline to complete and deploy the work.

### Task
I needed to deliver the implementation while ensuring that the permissions and business rules matched the stakeholder expectations.

### Action
I reviewed the available user stories and scope, and whenever I had questions about permissions, specifications, or expected behavior, I worked directly with the stakeholder team. I regularly clarified requirements, discussed design decisions, and kept stakeholders updated on the implementation. I also collaborated with the verification and QA teams to confirm that the access-control behavior matched the expected business rules.

### Result
Because of the ongoing communication, I was able to adapt to requirement changes and complete the implementation within the required timeline. We successfully deployed it to production without any major issues. I learned that cross-functional collaboration is especially important when requirements are complex or changing.

## 13. Tell me about a time you disagreed with your manager’s decision. What did you do?

### Situation
During testing, I found an issue involving the application’s translation library. When a translation ID was empty, the library caused a null-pointer exception and the page became unusable, directly affecting the user experience.

### Task
I needed to help resolve the immediate issue while also considering whether the same problem could exist on other pages.

### Action
I suggested creating a reusable component and applying it across all pages using the library. My reasoning was that this could prevent similar issues elsewhere. However, my manager and teammates were concerned that making changes across multiple pages could introduce new bugs and delay the release. Their preference was to fix the immediate issue first.

Rather than continuing the disagreement informally, I brought the manager, stakeholders, and relevant teammates together to discuss the options. We agreed on a hybrid approach: fix the affected page immediately and plan the reusable component for a future release after proper testing.

### Result
The immediate production risk was resolved without blocking delivery, while the broader technical improvement was preserved for a future release. I learned that disagreements are best handled by focusing on the problem, listening to other viewpoints, and finding a balanced solution.

## 14. Tell me about a time you had to give or receive difficult feedback.

### Situation
In one project, I helped onboard several junior developers. I guided them through the codebase, architecture, business domain, frontend and backend data flow, and the coding standards used in the repository.

### Task
After assigning them smaller stories, I needed to review their pull requests and help them improve their implementation quality.

### Action
Their implementations were generally functionally correct, but I noticed that they were not consistently following the project’s coding style, design patterns, object-oriented principles, or pull-request standards. I gave specific feedback in their pull requests and scheduled a discussion with them. I first acknowledged what they were doing well, including the correctness of their workflows, and then clearly explained the areas they needed to improve. I also showed them examples of the expected coding style and pull-request descriptions.

### Result
They improved significantly and began following the project standards more consistently. This taught me that difficult feedback is most effective when it is specific, respectful, and focused on helping the person grow.

## 15. Tell me about a time you had to balance competing priorities or deadlines.

### Situation
During one project, we were preparing a new environment for a client while also developing new features. At the same time, we had to break down stories, assign work to multiple developers, address pull-request review comments, and complete backend development and configuration tasks.

### Task
I needed to balance these responsibilities while keeping the client deployment on schedule.

### Action
I prioritized the work based on its impact on the release. First, I addressed the pull-request review comments because those changes were required before the features could be merged into the development branch and tested by stakeholders. Next, I focused on implementing the highest-priority features defined in the scope and created pull requests for internal review. Lower-priority backend tasks were scheduled alongside the main work or moved later so they would not affect the client deployment timeline.

### Result
By organizing the work according to urgency, dependencies, and stakeholder impact, we kept the release moving and ensured that the most important functionality was ready for testing. I learned to prioritize work based on business impact and dependencies rather than simply handling tasks in the order they arrive.

## 16. Where do you see yourself in the next few years?

### Situation
As my technical experience continues to grow, I want to take on broader responsibilities within engineering projects.

### Task
My goal is to develop both deeper technical expertise and stronger leadership capabilities.

### Action
I plan to take ownership of complex features, contribute to architectural and implementation decisions, guide teammates, and mentor junior developers. I also want to become more involved in coordinating work and helping teams deliver successfully.

### Result
Over the next few years, I would like to grow into a technical leadership role where I can contribute through my own engineering work and help the wider team succeed. I want to grow with the organization and take on responsibilities that create value for both the team and the product.

## 17. What kind of work environment or team culture do you thrive in?

### Situation
The projects I work on often involve complex features, changing requirements, and collaboration across different teams.

### Task
To deliver successfully, I need to work in an environment where people communicate clearly and support one another.

### Action
I thrive in a collaborative and communicative team where people feel comfortable asking questions, discussing ideas, and raising concerns early. If a teammate is blocked, I like to help them understand the issue and find a solution so the work can continue. I also believe disagreements should be handled through respectful discussion, with the focus on solving the problem rather than proving who is right.

### Result
In this type of culture, people share knowledge, give constructive feedback, and help each other grow. It creates an environment where the team can deliver work on time while continuing to learn and improve together.

## 18. Tell me about a time you failed at something. What did you learn?

### Situation
One failure I experienced was deploying a feature without testing one of its critical workflows thoroughly enough. The feature included CRUD operations, but I missed properly integrating the delete API on one page.

### Task
I was responsible for ensuring that the feature worked correctly before it was released.

### Action
After deployment, the delete functionality did not work for users. Once the issue was discovered, I took responsibility and informed my manager and the relevant stakeholders. I investigated the cause, fixed the API integration, tested the functionality, and supported the redeployment to production.

### Result
The functionality was restored, but the more important outcome was the lesson I gained. Release pressure should not reduce the quality of testing, especially for important user actions. Since then, I have been more disciplined about regression testing, end-to-end testing, and validating critical workflows before deployment.

## 19. How do you handle ambiguity or unclear requirements?

### Situation
In one project, I was responsible for implementing access control and file-share history features. The overall story was complex, and although the high-level design was available, many detailed specifications were unclear and continued to change during development.

### Task
I needed to make progress while ensuring that the implementation continued to match the current business requirements.

### Action
I reviewed the available scope carefully and stayed in consistent communication with the stakeholders. Whenever I had questions about the specifications, design, or expected behavior, I clarified them before continuing. I also kept stakeholders updated on my progress so changes could be identified early rather than after the entire implementation was complete. As the requirements evolved, I adjusted the implementation and continued validating the functionality with the stakeholders.

### Result
I completed and deployed the features successfully, and the stakeholders appreciated the outcome. I learned that unclear requirements should be handled through proactive communication, frequent validation, and flexibility rather than assumptions.

## 20. Given this is a senior role, tell me about a time you mentored or guided a junior teammate.

### Situation
Several junior developers joined one of my projects, and they needed support understanding the existing codebase, architecture, business domain, and development practices.

### Task
I was responsible for helping them become productive and guiding them as they began working on project stories.

### Action
I provided knowledge-transfer sessions covering the project workflow, frontend and backend data flow, coding style, and repository standards. I assigned smaller stories and bug fixes so they could learn through practical work. I reviewed their pull requests and gave constructive feedback on design patterns, object-oriented principles, coding style, and pull-request descriptions. I also explained the reasons behind the standards and showed examples of how to improve their work.

### Result
The junior developers improved their implementations, coding practices, and pull-request quality. They became more comfortable working in the codebase and were able to contribute more independently. This experience strengthened my ability to mentor others and showed me how clear guidance and feedback can accelerate a teammate’s development.



## General reminders before the interview

- Avoid filler phrases in delivery: "so yeah", "that's all", "so so".
- Every STAR answer lands harder with a **number** — time saved, % improvement, team size, deadline length. Fill in the TODOs above before interview day.
- This role (ODL team) is about the **data delivery/visualization layer** on top of Anaplan's core platform — not Anaplan's core Hyperblock planning engine. Be honest about which JD-listed tools you have experience with (React/TypeScript/Spring Boot — yes) vs. which you don't yet (Kotlin, Flink, Databricks, Kubernetes, observability tools) — and pivot to your proven ability to ramp up quickly (see Q4 and Q7 answers) if asked.
- Anaplan's 8 core values: **Innovative, Accountable, Collaborative, Transparent, Resilient, Empathetic, Authentic, Learner.** Many answers above map naturally to "Learner" (Q4, Q7), "Collaborative" (Q6, Q2), and "Accountable" (Q5, Q8).
