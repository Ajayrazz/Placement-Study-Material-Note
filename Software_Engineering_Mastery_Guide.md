# Software Engineering (SE) — The SDE-1 & SWE Intern Interview Handbook

> **Target Audience:** SWE Interns & SDE-1 Candidates  
> **Objective:** Master modern software engineering paradigms, Agile/Scrum, testing pyramid, CI/CD, deployment strategies, software architecture, technical debt, and real-world system delivery.

---

## Table of Contents
1. [Software Development Life Cycle (SDLC)](#1-software-development-life-cycle-sdlc)
   - [The Core Phases of SDLC](#the-core-phases-of-sdlc)
   - [Waterfall vs. Agile vs. V-Model vs. Spiral](#waterfall-vs-agile-vs-v-model-vs-spiral)
2. [Agile Methodology & Scrum Framework (High Frequency)](#2-agile-methodology--scrum-framework-high-frequency)
   - [The 4 Core Values & 12 Principles](#the-4-core-values--12-principles)
   - [Scrum Roles, Ceremonies, and Artifacts](#scrum-roles-ceremonies-and-artifacts)
   - [Story Points, Velocity, and Planning Poker](#story-points-velocity-and-planning-poker)
   - [Scrum vs. Kanban](#scrum-vs-kanban)
3. [Software Architecture & Design Paradigms](#3-software-architecture--design-paradigms)
   - [Monolithic vs. Microservices vs. Serverless](#monolithic-vs-microservices-vs-serverless)
   - [Layered (N-Tier) Architecture](#layered-n-tier-architecture)
   - [Event-Driven Architecture (EDA)](#event-driven-architecture-eda)
4. [Software Testing & Quality Assurance](#4-software-testing--quality-assurance)
   - [The Testing Pyramid (Unit vs. Integration vs. E2E)](#the-testing-pyramid)
   - [TDD (Test-Driven Development) & BDD](#tdd-test-driven-development--bdd)
   - [Test Doubles: Dummy vs. Stub vs. Spy vs. Mock vs. Fake](#test-doubles)
   - [Smoke, Sanity, Regression, and Stress Testing](#types-of-testing)
5. [CI/CD & DevOps Engineering](#5-cicd--devops-engineering)
   - [Continuous Integration vs. Continuous Delivery vs. Continuous Deployment](#continuous-integration-vs-delivery-vs-deployment)
   - [Git Branching Strategies (GitFlow vs. Trunk-Based Development)](#git-branching-strategies)
   - [Modern Deployment Strategies (Canary, Blue-Green, Rolling)](#modern-deployment-strategies)
6. [Code Quality, Clean Code & Technical Debt](#6-code-quality-clean-code--technical-debt)
   - [Core Principles: DRY, KISS, YAGNI, Law of Demeter](#core-clean-code-principles)
   - [Code Smells & Refactoring Patterns](#code-smells--refactoring-patterns)
   - [Technical Debt: Causes, Interest, and Repayment](#technical-debt)
   - [How to Conduct an SDE-Level Code Review](#how-to-conduct-an-sde-level-code-review)
7. [System Reliability & Observability (SRE Basics)](#7-system-reliability--observability-sre-basics)
   - [The 3 Pillars of Observability (Logs, Metrics, Traces)](#the-3-pillars-of-observability)
   - [SLA vs. SLO vs. SLI](#sla-vs-slo-vs-sli)
   - [Resilience Patterns: Circuit Breaker, Retries with Exponential Backoff](#resilience-patterns)
8. [Top 20 SDE-1 / Intern Software Engineering Interview Questions & Model Answers](#8-top-20-sde-1--intern-software-engineering-interview-questions--model-answers)

---

## 1. Software Development Life Cycle (SDLC)

### What is SDLC?
SDLC is a structured, systematic process for planning, creating, testing, deploying, and maintaining high-quality software systems.

```
       +--------------------------------------------------------+
       |                     SDLC PHASES                        |
       +--------------------------------------------------------+
                                   |
    [ 1. Requirements ] ---> Product Requirement Document (PRD) / SRS
            |
    [ 2. System Design ] --> High-Level Design (HLD) & Low-Level Design (LLD)
            |
    [ 3. Implementation ] -> Coding, Peer Code Reviews, Unit Tests
            |
    [ 4. Verification ] ---> Integration, Regression, Load & Security Testing
            |
    [ 5. Deployment ] -----> Staging, Canary Rollout, Production Release
            |
    [ 6. Maintenance ] ----> Monitoring, Bug Fixes, Scaling, Hotfixes
```

---

### Waterfall vs. Agile vs. V-Model vs. Spiral

```
+---------------------------------------------------------------------------------------------------+
|                                     SDLC MODELS COMPARISON                                        |
+-------------------+----------------------+--------------------+--------------------+--------------+
| Feature           | Waterfall            | Agile (Scrum)      | V-Model            | Spiral       |
+-------------------+----------------------+--------------------+--------------------+--------------+
| Approach          | Sequential / Linear  | Iterative / Incre- | Verification &     | Risk-driven  |
|                   |                      | mental             | Validation paired  | iterative    |
| Flexibility       | Rigid. Changes are   | High. Changes      | Low. Testing phases| High. Cycles |
| to Change         | very costly.         | welcomed anytime.  | planned early.     | assess risks.|
| Customer Feedback | End of project       | Every 1-2 weeks    | End of project     | After every  |
|                   | (Late discovery)     | (Continuous demo)  |                    | spiral cycle |
| Risk Management   | Poor (Disasters found| High (Immediate    | Medium (Caught by  | **Highest**  |
|                   | during integration)  | course-correction) | early test plans)  | (Risk checks)|
| Best For          | Defense, NASA, fixed | Web apps, startups,| Healthcare devices,| High-risk,   |
|                   | government contracts | SaaS products      | automotive, avionics| massive R&D |
+-------------------+----------------------+--------------------+--------------------+--------------+
```

> **Real-World Analogy:**
> - **Waterfall:** Constructing a 50-story concrete skyscraper. You cannot alter the basement layout after reaching the 40th floor.
> - **Agile:** Launching an e-commerce startup. You launch an MVP landing page with 10 products, collect user checkout feedback, and ship improvements every 2 weeks.

---

## 2. Agile Methodology & Scrum Framework (High Frequency)

### The 4 Core Values of the Agile Manifesto
1. **Individuals and interactions** over processes and tools.
2. **Working software** over comprehensive documentation.
3. **Customer collaboration** over contract negotiation.
4. **Responding to change** over following a rigid plan.

---

### Scrum Roles, Ceremonies, and Artifacts

```
                       THE SCRUM FRAMEWORK CYCLE
                       
Product Backlog       Sprint Planning       Daily Standup (15 min)
 [ Feature A ]            |                   (What did I do yesterday?
 [ Feature B ] ---------> v                    What will I do today?
 [ Feature C ]     Sprint Backlog              Any blockers?)
                     (2-Week Sprint)
                            |                          |
                            +--------------------------+
                            |
                            v
                   Potentially Shippable Product Increment
                            |
                 +----------+----------+
                 v                     v
          Sprint Review        Sprint Retrospective
      (Demo to stakeholders) (Team looks back: What went well?
                              What went wrong? Action items.)
```

#### 1. The 3 Scrum Roles
- **Product Owner (PO):** Represents the business and customer. Owns and prioritizes the **Product Backlog** based on ROI and business value.
- **Scrum Master (SM):** Servant-leader. Removes external blockers for developers, facilitates ceremonies, and enforces Agile hygiene. *Not a project manager!*
- **Developers / Engineers:** Cross-functional team (SDEs, QAs, Designers) that self-organizes to execute and deliver the sprint commitment.

#### 2. The 5 Scrum Ceremonies
- **Sprint Planning:** Team commits to a set of User Stories from the top of the Product Backlog for the next sprint (typically 2 weeks).
- **Daily Standup (Scrum):** 15-minute daily synchronization answering 3 questions:
  1. *What did I accomplish yesterday?*
  2. *What will I work on today?*
  3. *Are there any blockers preventing progress?*
- **Sprint Review:** End-of-sprint demo showing live, working software to stakeholders.
- **Sprint Retrospective:** Internal engineering reflection on process, team dynamics, and technical friction to improve the next sprint.
- **Backlog Refinement (Grooming):** Breaking large Epics into User Stories, defining Acceptance Criteria, and estimating story points.

---

### Story Points, Velocity, and Planning Poker
- **Story Point:** A relative unit of measure representing the **effort, complexity, and uncertainty** of a task (not direct hours!). Usually uses the Fibonacci sequence ($1, 2, 3, 5, 8, 13, 21$).
- **Planning Poker:** Consensus-based estimation technique where engineers simultaneously reveal card estimates to avoid anchoring bias.
- **Velocity:** The average number of story points an engineering team completes per sprint. Used to forecast delivery timelines.

---

### Scrum vs. Kanban

| Dimension | Scrum | Kanban |
| :--- | :--- | :--- |
| **Cadence** | Fixed-length iterations (e.g., 2-week Sprints). | Continuous flow (no sprints; pull items as capacity opens). |
| **Metrics** | Velocity, Sprint Burndown Chart. | Cycle Time, Lead Time, Cumulative Flow Diagram. |
| **Core Constraint** | Commitment to Sprint scope (no changes mid-sprint). | **WIP Limits (Work In Progress):** Caps max tasks in a column. |
| **Best For** | Product feature development teams. | DevOps, SRE, Production Support, Bug Triaging teams. |

---

## 3. Software Architecture & Design Paradigms

### Monolithic vs. Microservices vs. Serverless

```
MONOLITHIC ARCHITECTURE                 MICROSERVICES ARCHITECTURE
+---------------------------------+     +-------------+    +-------------+
| UI / Presentation Layer         |     | Auth Service|    |Order Service|
|---------------------------------|     | (Postgres)  |    | (MongoDB)   |
| Business / Domain Logic         |     +-------------+    +-------------+
|---------------------------------|            \                 /
| Data Access Layer (Single DB)   |             v               v
+---------------------------------+        [ API Gateway / Envoy ]
                                                   |
                                            +-------------+
                                            |Pay Service  |
                                            | (Redis/SQL) |
                                            +-------------+
```

| Dimension | Monolithic | Microservices | Serverless (FaaS) |
| :--- | :--- | :--- | :--- |
| **Deployment** | Single deployable binary / artifact (`.jar`, `.war`, container). | Dozens/hundreds of independently deployed services. | Ephemeral cloud functions (AWS Lambda, Google Cloud Functions). |
| **Scalability** | Scale the whole app or nothing (horizontal duplication). | Scale only bottlenecks (e.g., scale Payment service 10x). | Auto-scales from 0 to thousands instantly per request. |
| **Failure Blast Radius**| Single memory leak or infinite loop crashes the **entire app**. | Isolated. If Recommendation service dies, Checkout still works. | Isolated to single function invocation. |
| **Operational Cost** | Simple (single CI/CD pipeline, local debugging easy). | High complexity (Distributed tracing, Kubernetes, network latencies). | Low infra management; cold start latency challenges. |

---

### Layered (N-Tier) Architecture
The industry-standard architectural style for clean backend engineering (Spring Boot, NestJS, Django, ASP.NET):

```
[ HTTP Client / Frontend ]
          |  (JSON / REST / gRPC Request)
          v
+-----------------------------------------------------------+
| 1. CONTROLLER LAYER (Presentation / API Gateway)          |
|    - Handles routing, HTTP status codes, request DTO      |
|      validation, parameter parsing                        |
+-----------------------------------------------------------+
          |
          v
+-----------------------------------------------------------+
| 2. SERVICE LAYER (Business / Domain Logic)                |
|    - Core business rules, transaction boundaries (@Transactional),|
|      orchestrating multiple repositories, calculations    |
+-----------------------------------------------------------+
          |
          v
+-----------------------------------------------------------+
| 3. DATA ACCESS LAYER (Repository / DAO)                   |
|    - Interacts directly with database via ORM / SQL       |
|    - Isolates DB schema changes from business logic       |
+-----------------------------------------------------------+
          |
          v
    [ DATABASE (PostgreSQL / MySQL / Redis) ]
```

---

## 4. Software Testing & Quality Assurance

### The Testing Pyramid

```
                /\
               /  \
              /    \
             / E2E  \       Slowest, Most Fragile, Highest Cost
            /--------\
           / Integr-  \     Tests interaction between components (DB, APIs)
          /   ation    \
         /--------------\
        /  Unit Tests    \  Fastest, Deterministic, Lowest Cost,
       /__________________\ Runs in milliseconds on local machine
```

1. **Unit Tests (Base):** Tests a single function, method, or class in absolute isolation. All external dependencies (DB, network, disk) are **mocked**. Runs in milliseconds.
2. **Integration Tests (Middle):** Verifies that multiple components work together properly (e.g., testing that `UserRepository` actually writes and reads from a test Docker PostgreSQL container).
3. **End-to-End (E2E) Tests (Peak):** Simulates real user behavior across the entire stack (e.g., Cypress/Selenium clicking login, adding items to cart, and checking payment confirmation).

---

### TDD (Test-Driven Development) & BDD

#### The TDD "Red-Green-Refactor" Cycle
```
          [ RED ] --------> Write a unit test that fails (compilation or assertion)
             ^                     |
             |                     v
         [ REFACTOR ] <---- [ GREEN ]
    Clean up code, remove     Write the MINIMAL production code
    duplication without       necessary to make the test pass
    breaking tests
```

#### BDD (Behavior-Driven Development)
Focuses on user behaviors described in human-readable DSL (**Gherkin syntax**):
```gherkin
Feature: User Checkout
  Scenario: Successful purchase with sufficient funds
    Given the user has $100 in their account balance
    When the user buys an item costing $40
    Then $40 should be deducted from their balance
    And the user should see an order confirmation message
```

---

### Test Doubles: Dummy vs. Stub vs. Spy vs. Mock vs. Fake
*Interviewers love asking the exact distinction between these:*

```
+--------------------------------------------------------------------------+
|                         TEST DOUBLES TAXONOMY                            |
+-------------------+------------------------------------------------------+
| Double Type       | How it Works & Purpose                               |
+-------------------+------------------------------------------------------+
| **Dummy**         | Passed around just to satisfy parameter lists; never |
|                   | actually executed or inspected (e.g., `null` or empty|
|                   | object).                                             |
| **Stub**          | Provides canned/hardcoded answers to method calls    |
|                   | during the test (e.g., `when(repo.get()).thenReturn(user)`).|
| **Spy**           | A real object that silently records execution info   |
|                   | (e.g., how many times a method was called).          |
| **Mock**          | Pre-programmed with expectations; verifies **behavior|
|                   | and interactions** (e.g., assert `sendEmail()` was   |
|                   | invoked exactly 1 time with target address).         |
| **Fake**          | Has a working implementation, but takes a shortcut   |
|                   | unsuitable for production (e.g., In-Memory H2 DB).   |
+-------------------+------------------------------------------------------+
```

---

### Types of Testing

- **Smoke Testing ("Build Verification Test"):** High-level check to confirm that the critical path works before running deep tests (e.g., Does the server start? Can the login page load?).
- **Sanity Testing:** Quick focused test to verify that a specific bug fix works and didn't break basic adjacent functionality.
- **Regression Testing:** Re-running test suites across the application to verify that newly added code hasn't broken existing, working features.
- **Stress vs. Load Testing:**
  * **Load Testing:** Tests system performance under expected normal and peak traffic loads.
  * **Stress Testing:** Pushes the system beyond its breaking point to see how gracefully it fails (does it crash completely or degrade gracefully?).

---

## 5. CI/CD & DevOps Engineering

### Continuous Integration vs. Delivery vs. Deployment

```
Developers Commit Code
         |
         v
[ CONTINUOUS INTEGRATION (CI) ]
- Automated Build
- Linting & Static Code Analysis (SonarQube)
- Unit & Integration Test Suites Run
         |
         v
[ CONTINUOUS DELIVERY (CD) ]
- Build Artifact Created (Docker Image / Jar)
- Automatically Deployed to Staging / QA Environment
- Automated Smoke & E2E Tests Pass
         |
         +----------------------------------+
         |                                  |
Manual Approval (1-Click)          Fully Automated Release
         |                                  |
         v                                  v
[ PRODUCTION DEPLOYMENT ]        [ CONTINUOUS DEPLOYMENT ]
```

---

### Git Branching Strategies

#### 1. GitFlow (Traditional / Complex)
- Uses long-lived branches: `main` (production), `develop` (staging), `feature/*`, `release/*`, and `hotfix/*`.
- **Downside:** Complex merge conflicts, slow releases; largely abandoned by modern cloud-native tech companies.

#### 2. Trunk-Based Development (Modern Industry Standard)
- All engineers merge small, frequent commits into a single shared branch (`main` / `trunk`) multiple times a day.
- Features under development are protected using **Feature Flags (Feature Toggles)**.
- **Advantage:** Eliminates "Merge Hell", enables true continuous integration, fast iterations.

---

### Modern Deployment Strategies

#### 1. Rolling Deployment
- Instances running the old version are updated to the new version one by one (or in batches) behind a load balancer.
- *Pros:* Zero downtime; requires no extra infrastructure resources.
- *Con:* For a period of time, both Old and New versions run concurrently (requires backward-compatible DB schemas!).

```
Step 1: [ v1 ] [ v1 ] [ v1 ] [ v1 ]
Step 2: [ v2 ] [ v1 ] [ v1 ] [ v1 ]
Step 3: [ v2 ] [ v2 ] [ v1 ] [ v1 ]
Step 4: [ v2 ] [ v2 ] [ v2 ] [ v2 ]
```

#### 2. Blue-Green Deployment
- Maintain two identical production environments: **Blue** (active live traffic) and **Green** (idle new release).
- Deploy v2 to Green, run sanity tests.
- Switch the Load Balancer router instantly from Blue to Green.
- *Pros:* Instant rollback (switch router back to Blue if bugs appear); zero mixed-version window.
- *Con:* Doubles infrastructure costs during deployment.

```
                  Router / Load Balancer
                       /         \
                      v           v
           [ Active: BLUE v1 ]   [ Staging: GREEN v2 ]
```

#### 3. Canary Deployment (Gold Standard for Big Tech)
- Route a small fraction of real production user traffic (e.g., 2% or internal employees) to the new version (`v2`), while 98% remain on `v1`.
- Monitor telemetry (error rates, CPU latency, crash reports).
- Gradually ramp up: $2\% \rightarrow 10\% \rightarrow 50\% \rightarrow 100\%$.
- *Pros:* Minimizes blast radius of critical production bugs.

---

## 6. Code Quality, Clean Code & Technical Debt

### Core Clean Code Principles

1. **DRY (Don't Repeat Yourself):** Every piece of knowledge or business logic must have a single, unambiguous representation in a system.
2. **KISS (Keep It Simple, Stupid):** Avoid over-engineering. The simplest solution that solves the problem correctly is superior to clever, complex code.
3. **YAGNI (You Aren't Gonna Need It):** Never implement functionality based on speculation of future needs. Build only what is needed today.
4. **Law of Demeter ("Principle of Least Knowledge"):** An object should only talk to its immediate neighbors (avoid method chaining like `order.getCustomer().getAddress().getZipCode()`).

---

### Code Smells & Refactoring Patterns

- **God Object (Large Class):** A class that knows too much or does too much. *Fix:* Apply Single Responsibility Principle (SRP) and extract smaller delegate classes.
- **Long Method:** A method spanning 50+ lines doing multiple tasks. *Fix:* Extract Method refactoring into cleanly named helper functions.
- **Feature Envy:** A method in Class A that makes more calls to getters/data of Class B than its own data. *Fix:* Move the method into Class B.
- **Shotgun Surgery:** Making a single business change requires modifying dozens of tiny files across the codebase. *Fix:* Consolidate related logic into a single cohesive module.

---

### Technical Debt

```
Technical Debt = "Shipping quick & dirty code today to meet a deadline, 
                  which incurs interest tomorrow in the form of slower 
                  development, fragile bugs, and painful maintenance."
```

- **Prudent Debt (Strategic):** Deliberately cutting corners to hit a critical market window, with an explicit plan to refactor in the next sprint.
- **Reckless Debt (Sloppy):** Inexperienced engineers writing messy code without unit tests or architecture reviews.
- **How to manage debt:** Dedicate 15–20% of engineering bandwidth every sprint to refactoring and debt repayment.

---

### How to Conduct an SDE-Level Code Review
When an interviewer asks: *"What do you look for when reviewing a pull request?"*

1. **Correctness & Edge Cases:** Does the code satisfy the acceptance criteria? Are null pointers, empty collections, and boundary numbers handled?
2. **Design & SOLID Principles:** Does it fit existing architectural patterns? Is there unnecessary coupling?
3. **Test Coverage:** Are unit tests included? Do they test failure modes as well as the happy path?
4. **Performance & Security:** Are there SQL injection vulnerabilities? Unindexed queries ($O(N^2)$ loops)? Race conditions in concurrent threads?
5. **Readability & Maintainability:** Are variables and methods named descriptively? Is code self-documenting without relying on misleading comments?

---

## 7. System Reliability & Observability (SRE Basics)

### The 3 Pillars of Observability

```
                       THE 3 PILLARS OF OBSERVABILITY
                                     |
        +----------------------------+----------------------------+
        |                            |                            |
    1. LOGS                      2. METRICS                   3. TRACES
  Discrete, timestamped        Numeric aggregations over    End-to-end request journey
  events with context          time intervals               across distributed microservices
  (JSON strings)               (Counters, Gauges, Histograms)(Distributed Tracing with TraceID)
  e.g., "User 42 payment       e.g., CPU Usage: 84%,        e.g., Frontend (12ms) -> Auth (4ms)
  failed: Insufficient Funds"   Error Rate: 0.02%            -> DB Query (180ms - BOTTLENECK!)
```

---

### SLA vs. SLO vs. SLI

| Term | Full Form | Meaning & Perspective | Example |
| :--- | :--- | :--- | :--- |
| **SLI** | Service Level Indicator | The actual **metric** measured right now. | "Our API successful response rate over the last 30 days is 99.93%." |
| **SLO** | Service Level Objective | The **internal engineering target** team strives for. | "Our target uptime must be at least 99.9%." |
| **SLA** | Service Level Agreement | The **legal/financial contract** with customers. Penalties paid if breached. | "If uptime drops below 99.5%, customers receive a 20% billing credit." |

---

### Resilience Patterns

#### 1. Circuit Breaker Pattern (Netflix Hystrix / Resilience4j)
Prevents cascading failures when a downstream microservice is down or unresponsive.

```
       [ Normal Requests ]
                |
                v
       +-----------------+
       |     CLOSED      |  (All calls pass through normally)
       +-----------------+
                | Failures exceed threshold (e.g. 50% timeout)
                v
       +-----------------+
       |      OPEN       |  (Fails FAST immediately without calling downstream;
       +-----------------+   returns fallback response / cached data)
                | After sleep window (e.g. 30 seconds)
                v
       +-----------------+
       |    HALF-OPEN    |  (Sends trial requests to test downstream health)
       +-----------------+
         /             \
 (Calls succeed)     (Calls fail)
       /                 \
      v                   v
   CLOSED                OPEN
```

#### 2. Retries with Exponential Backoff & Jitter
- When a network request fails, do **not** hammer the downstream server immediately in a tight loop.
- **Exponential Backoff:** Wait $100\text{ms} \rightarrow 200\text{ms} \rightarrow 400\text{ms} \rightarrow 800\text{ms}$.
- **Jitter:** Add random noise (e.g., $\pm 25\text{ms}$) to prevent the **Thundering Herd problem** where thousands of clients retry at the exact same millisecond.

---

## 8. Top 20 SDE-1 / Intern Software Engineering Interview Questions & Model Answers

### Q1: What is the difference between Agile and Waterfall? When would you use Waterfall?
**Answer:** Waterfall is a linear, sequential model where each phase (requirements, design, coding, testing) must complete before the next begins; it is rigid to change but offers predictability. Agile is iterative and incremental, delivering working software in short cycles (sprints) with continuous customer feedback. You would use Waterfall for projects with strictly fixed, unchangeable requirements and life-critical compliance (e.g., pacemaker firmware, aerospace flight controllers, defense contracts).

---

### Q2: What is the purpose of a Daily Standup in Scrum? What 3 questions do you answer?
**Answer:** The daily standup is a 15-minute synchronization ceremony for developers to align on sprint progress and expose blockers. It is not a status report to management. Every member answers: (1) What did I complete yesterday? (2) What do I plan to work on today? (3) Are there any impediments or blockers in my way?

---

### Q3: What is the difference between a Mock and a Stub in unit testing?
**Answer:** A Stub is a test double that provides predefined, canned return values to method calls made during a test without checking how it was called. A Mock is pre-configured with expectations about method invocations; it verifies behavior and interactions (e.g., asserting that `emailService.send()` was called with specific arguments exactly once).

---

### Q4: Explain the Testing Pyramid. Why should an organization have more Unit Tests than End-to-End tests?
**Answer:** The Testing Pyramid dictates that test suites should be composed mostly of Unit Tests at the base, followed by Integration Tests, with End-to-End (E2E) tests at the top. Unit tests are fast (milliseconds), cheap to write, deterministic, and pinpoint the exact line of failure. E2E tests are slow, computationally expensive, brittle to UI changes, and suffer from flakiness. Having more unit tests provides fast developer feedback and lower maintenance overhead.

---

### Q5: What is the difference between Continuous Delivery and Continuous Deployment?
**Answer:** Both automate the build, linting, testing, and staging deployment phases. The difference is the final step into production: in **Continuous Delivery**, releasing to production requires manual human authorization (a one-click approval). In **Continuous Deployment**, every single commit that passes all automated CI/CD pipeline tests is automatically deployed to live production without any human intervention.

---

### Q6: What is a Canary Deployment? What problem does it solve?
**Answer:** A Canary Deployment routes a small fraction of real production traffic (e.g., 2%) to a new version of the software while the remaining 98% continue using the stable version. It solves the risk of catastrophic outages by limiting the blast radius of unexpected production bugs, allowing engineers to verify real telemetry (error rates, latency) before rolling out to 100% of users.

---

### Q7: What is Technical Debt? How do you explain it to non-technical stakeholders?
**Answer:** Technical debt is a metaphor representing the future cost of rework caused by choosing an easy, fast solution today instead of a better, scalable design. Just like financial debt, it incurs "interest": future development takes longer because engineers must work around messy, fragile code. It should be repaid through periodic refactoring before the interest grinds feature delivery to a halt.

---

### Q8: What is the difference between GitFlow and Trunk-Based Development?
**Answer:** GitFlow relies on multiple long-lived branches (`main`, `develop`, feature branches, release branches) that merge infrequently, often causing painful merge conflicts. Trunk-Based Development requires all developers to merge small, frequent commits into a single shared branch (`trunk`/`main`) daily, using feature flags to hide incomplete functionality. Trunk-based development enables true continuous integration and faster feedback.

---

### Q9: What is the Circuit Breaker pattern in microservices?
**Answer:** A Circuit Breaker protects a system from cascading failures when calling an external or downstream service. It operates in 3 states: **Closed** (requests flow normally), **Open** (tripped by high error/timeout rates; calls fail immediately without hitting the downstream service, returning a fallback), and **Half-Open** (periodically tests if the downstream service has recovered before restoring normal traffic).

---

### Q10: What is the difference between Black Box, White Box, and Grey Box testing?
**Answer:**
- **Black Box:** Tester has no knowledge of internal code or architecture; tests strictly against inputs and outputs based on requirements.
- **White Box:** Tester has full access to the source code, internal data structures, and algorithms; tests code paths, branches, and statement coverage.
- **Grey Box:** Tester has partial knowledge of internals (e.g., understands DB schema or API contracts, but does not read full implementation code).

---

### Q11: What is the difference between Monolith and Microservices? What is the biggest drawback of Microservices?
**Answer:** A monolith is a single unified application where all business domains share the same codebase and database. Microservices decompose the system into small, independently deployable services organized around business capabilities, communicating via APIs/message buses. The biggest drawback of microservices is **operational and distributed system complexity**: network latency, distributed data consistency (lack of ACID transactions across services), distributed debugging/tracing, and deployment overhead.

---

### Q12: What is the DRY principle? Can DRY ever be taken too far?
**Answer:** DRY ("Don't Repeat Yourself") states that every piece of knowledge must have a single representation in a system, avoiding copy-pasting logic. It can be taken too far when two unrelated pieces of code happen to look identical by coincidence; forcing them into a shared abstraction creates **tight coupling** where future modifications to one use case break the other. As the saying goes, *"Duplication is far cheaper than the wrong abstraction."*

---

### Q13: What is the difference between Smoke Testing and Sanity Testing?
**Answer:** Smoke Testing is shallow and broad; it verifies that the most critical functions work at a high level to ensure the build is stable enough for deeper testing. Sanity Testing is narrow and deep; it is performed after a minor bug fix to verify that the specific problem was resolved and that closely related components weren't broken.

---

### Q14: What is the purpose of Feature Flags (Feature Toggles)?
**Answer:** Feature flags allow software features to be enabled or disabled dynamically at runtime without deploying new code. They decouple code deployment from feature release, enabling dark launches, A/B testing, Canary rollouts, and instant kill-switches if a feature causes production issues.

---

### Q15: What is the difference between SLA, SLO, and SLI?
**Answer:** An **SLI (Indicator)** is the actual real-time measurement of service behavior (e.g., current latency). An **SLO (Objective)** is the internal reliability target set by the engineering team (e.g., latency must be $< 100\text{ms}$ for $99.9\%$ of requests). An **SLA (Agreement)** is the external legal contract with customers detailing penalties or refunds if performance falls below a minimum threshold.

---

### Q16: What is Blue-Green deployment? What is its main drawback?
**Answer:** Blue-Green deployment maintains two identical production environments: Blue (active live traffic) and Green (new version being deployed and tested). Once verified, the load balancer switches traffic from Blue to Green instantly. Its main drawback is **cost and resource overhead**, as it requires maintaining double the server capacity during the deployment process.

---

### Q17: What does "Shift-Left" mean in the software development lifecycle?
**Answer:** "Shift-Left" refers to moving critical software practices (testing, security vulnerability scans, performance audits) earlier in the SDLC timeline (to the "left" on a timeline diagram). By running automated linting, unit tests, and security scans during coding rather than right before production release, bugs are caught when they are cheapest and easiest to fix.

---

### Q18: What is the Law of Demeter? Give an example of a violation.
**Answer:** The Law of Demeter ("Principle of Least Knowledge") states that a method should only call methods on its own object, its parameters, or objects it creates directly. An example of a violation is:
```java
// Violation: Deeply chained navigation exposing internal object graphs
customer.getWallet().getCreditCard().deduct(amount);

// Correct: Ask the customer directly
customer.pay(amount);
```

---

### Q19: What is the difference between Regression Testing and Retesting?
**Answer:** Retesting is testing a specific defect that was previously reported to confirm that the developer's bug fix resolved it. Regression Testing is testing the **rest of the application** to ensure that the bug fix or new feature did not inadvertently introduce side effects or break unrelated existing functionality.

---

### Q20: What is Exponential Backoff with Jitter in network retries?
**Answer:** When an API request fails, retrying immediately can overwhelm a struggling server. Exponential backoff increases the wait time exponentially with each retry attempt ($100\text{ms}, 200\text{ms}, 400\text{ms}$). Jitter adds random variation to the wait times to prevent all synchronized clients from retrying at the exact same moment, avoiding the **Thundering Herd problem**.
