<div align="center">

# Software Testing · Course Projects

**University of Tehran — Software Testing (SWT), Fall 1403 / 2024**
Instructor: Dr. Ehsan Khamespanah

*Six assignments that walk one codebase from a first `assertEquals` all the way to
recorded browser scenarios — stopping at mocks, control-flow graphs, logic criteria,
mutants and a CI pipeline along the way.*

<br/>

![Java](https://img.shields.io/badge/Java_21-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white)
![Spring Boot](https://img.shields.io/badge/Spring_Boot-6DB33F?style=for-the-badge&logo=springboot&logoColor=white)
![JUnit5](https://img.shields.io/badge/JUnit_5-25A162?style=for-the-badge&logo=junit5&logoColor=white)
![Maven](https://img.shields.io/badge/Maven-C71A36?style=for-the-badge&logo=apachemaven&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/CI-2088FF?style=for-the-badge&logo=githubactions&logoColor=white)

![Mockito](https://img.shields.io/badge/Mockito-78A641?style=for-the-badge)
![JaCoCo](https://img.shields.io/badge/JaCoCo-4A5568?style=for-the-badge)
![PIT](https://img.shields.io/badge/PIT_mutation-E8590C?style=for-the-badge)
![Selenium](https://img.shields.io/badge/Selenium-43B02A?style=for-the-badge&logo=selenium&logoColor=white)
![Cucumber](https://img.shields.io/badge/Cucumber-23D96C?style=for-the-badge&logo=cucumber&logoColor=white)

</div>

---

## The arc

Each assignment adds one layer of testing theory on top of the last. Same systems, deeper questions.

```mermaid
flowchart LR
    A["CA1<br/>Unit Testing<br/>JUnit 5"]
    B["CA2<br/>Test Doubles<br/>Mockito"]
    C["CA3<br/>Graph Coverage<br/>JaCoCo"]
    D["CA4<br/>Logic &amp; ISP<br/>MockMvc"]
    E["CA5<br/>Mutation + CI<br/>PIT · Actions"]
    F["CA6<br/>GUI &amp; BDD<br/>Katalon · Cucumber"]

    A --> B --> C --> D --> E --> F

    classDef s1 fill:#3B5BDB,stroke:#1B2F7A,stroke-width:0px,color:#ffffff
    classDef s2 fill:#5F3DC4,stroke:#2E1C66,stroke-width:0px,color:#ffffff
    classDef s3 fill:#0B7285,stroke:#063844,stroke-width:0px,color:#ffffff
    classDef s4 fill:#087F5B,stroke:#04402E,stroke-width:0px,color:#ffffff
    classDef s5 fill:#E8590C,stroke:#7A2D06,stroke-width:0px,color:#ffffff
    classDef s6 fill:#C2255C,stroke:#63122F,stroke-width:0px,color:#ffffff

    class A s1
    class B s2
    class C s3
    class D s4
    class E s5
    class F s6
```

| | Assignment | What is being tested | Core idea | Tooling | Tests |
|:--|:--|:--|:--|:--|--:|
| ![](https://img.shields.io/badge/CA1-3B5BDB?style=flat-square) | **[Unit Testing](CA1)** | Mizdooni domain model | isolated units, parameterized cases | JUnit 5 | 36 |
| ![](https://img.shields.io/badge/CA2-5F3DC4?style=flat-square) | **[Test Doubles](CA2)** | Mizdooni controllers | mock vs. stub, state vs. behavior verification | Mockito | 58 |
| ![](https://img.shields.io/badge/CA3-0B7285?style=flat-square) | **[Graph Coverage](CA3)** | Fesadyab fraud engine | CFG, statement & branch coverage | JaCoCo | 14 |
| ![](https://img.shields.io/badge/CA4-087F5B?style=flat-square) | **[Logic & ISP](CA4)** | Mizdooni HTTP API | CACC / RACC, input-space partitioning | MockMvc | 83 |
| ![](https://img.shields.io/badge/CA5-E8590C?style=flat-square) | **[Mutation & CI](CA5)** | Fesadyab fraud engine | mutants as a test of the tests | PIT + GitHub Actions | 14 |
| ![](https://img.shields.io/badge/CA6-C2255C?style=flat-square) | **[GUI & BDD](CA6)** | Mizdooni through the browser | record-and-replay, Given/When/Then | Katalon Recorder · Selenium · Cucumber | 83 <sub>+8 scenarios</sub> |

> Every assignment folder is a self-contained snapshot of the project at that point, so later
> folders carry the earlier suites forward — 288 test methods across the six snapshots.

---

## The two systems under test

<table>
<tr>
<td width="50%" valign="top">

### 🍽 Mizdooni
A Spring Boot restaurant-reservation service: users sign up, browse restaurants,
check which tables are free in a time window, reserve, review and cancel.
Rich domain invariants (working hours, manager permissions, duplicate reviews,
past datetimes) make it a generous target for tests.

**Used in** CA1 · CA2 · CA4 · CA6

</td>
<td width="50%" valign="top">

### 🕵️ Fesadyab
A compact bank-fraud detector: it keeps a transaction history, derives per-account
averages, hunts for arithmetic patterns above a threshold, and scores suspicious
debits. Tiny surface, dense branching — exactly what coverage and mutation
analysis want.

**Used in** CA3 · CA5

</td>
</tr>
</table>

```mermaid
flowchart LR
    U["User"] -->|places| R["Reservation"]
    R -->|occupies| T["Table"]
    T -->|belongs to| RS["Restaurant"]
    U -->|writes| RV["Review"]
    RV -->|aggregates into| RT["Rating<br/>food · service · ambiance · overall"]
    RT -->|star count| RS
    M["Manager"] -->|owns| RS

    classDef core fill:#3B5BDB,stroke:#1B2F7A,stroke-width:0px,color:#ffffff
    classDef sat  fill:#E7ECFB,stroke:#3B5BDB,stroke-width:1px,color:#101828
    class U,RS core
    class R,T,RV,RT,M sat
```

---

<a id="ca1"></a>
## ![](https://img.shields.io/badge/CA1-3B5BDB?style=flat-square) Unit Testing · JUnit 5

> **The problem.** Mizdooni ships with no tests at all. Build the first safety net: unit tests for
> the logic-carrying methods of `User`, `Table`, `Restaurant` and `Rating` — one behaviour per test,
> no shared state between tests, and no wasted tests on bare getters, setters or constructors.

![](https://img.shields.io/badge/unit_testing-3B5BDB?style=flat-square&labelColor=1b1f24)
![](https://img.shields.io/badge/test_isolation-3B5BDB?style=flat-square&labelColor=1b1f24)
![](https://img.shields.io/badge/fixtures_@BeforeEach-3B5BDB?style=flat-square&labelColor=1b1f24)
![](https://img.shields.io/badge/parameterized_tests-3B5BDB?style=flat-square&labelColor=1b1f24)
![](https://img.shields.io/badge/@CsvSource-3B5BDB?style=flat-square&labelColor=1b1f24)
![](https://img.shields.io/badge/boundary_values-3B5BDB?style=flat-square&labelColor=1b1f24)
![](https://img.shields.io/badge/assume_vs_assert-3B5BDB?style=flat-square&labelColor=1b1f24)

The interesting cases are the ones where the domain quietly bends the rules — a cancelled
reservation that must stop counting as reserved, a second review by the same user that should
*replace* rather than append, an empty restaurant whose average rating must not divide by zero.
`Rating.getStarCount()` is the natural parameterized target: a rounding rule with a hard ceiling
at five stars, swept across eight inputs instead of eight near-identical test methods.

```java
@ParameterizedTest
@CsvSource({
    "4.6, 5",   // rounds up, clamped at the maximum
    "2.5, 3",   // half rounds up
    "0.1, 0",   // rounds down to the floor
    "10.0, 5"   // out of range, still capped
})
public void testGetStarCount(double overall, int expectedStarCount) { ... }
```

**Where to look** → [`CA1/Mizdooni/src/test/java/mizdooni/model/`](CA1/Mizdooni/src/test/java/mizdooni/model) ·
theory answers in [`CA1/Theory-Questions.md`](CA1/Theory-Questions.md)

---

<a id="ca2"></a>
## ![](https://img.shields.io/badge/CA2-5F3DC4?style=flat-square) Test Doubles · Mockito

> **The problem.** Controllers cannot be unit-tested honestly while they reach into real services.
> Write genuine *unit* tests for `AuthenticationController`, `ReservationController` and
> `ReviewController` by replacing every collaborator with a test double — and fix the small bugs
> the tests expose along the way.

![](https://img.shields.io/badge/mock_objects-5F3DC4?style=flat-square&labelColor=1b1f24)
![](https://img.shields.io/badge/stubs_&_spies-5F3DC4?style=flat-square&labelColor=1b1f24)
![](https://img.shields.io/badge/state_verification-5F3DC4?style=flat-square&labelColor=1b1f24)
![](https://img.shields.io/badge/behavior_verification-5F3DC4?style=flat-square&labelColor=1b1f24)
![](https://img.shields.io/badge/mockist_vs_classical-5F3DC4?style=flat-square&labelColor=1b1f24)
![](https://img.shields.io/badge/@Mock_%C2%B7_@InjectMocks-5F3DC4?style=flat-square&labelColor=1b1f24)
![](https://img.shields.io/badge/exception_paths-5F3DC4?style=flat-square&labelColor=1b1f24)
![](https://img.shields.io/badge/shared_vs_fresh_fixture-5F3DC4?style=flat-square&labelColor=1b1f24)

Mockito draws a hard border around the controller: everything inside is real code under test,
everything outside is a double that is either *trained* (`when(...).thenReturn(...)`) or
*interrogated* (`verify(...)`).

```mermaid
flowchart LR
    TEST["ReservationControllerTest"]
    CTRL["ReservationController<br/><i>system under test</i>"]
    S1["ReservationService"]
    S2["RestaurantService"]

    TEST -->|"@InjectMocks"| CTRL
    CTRL --> S1
    CTRL --> S2
    TEST -. "@Mock · when / thenReturn / thenThrow" .-> S1
    TEST -. "verify · times · never" .-> S2

    classDef sut fill:#5F3DC4,stroke:#2E1C66,stroke-width:0px,color:#ffffff
    classDef dbl fill:#EFE9FC,stroke:#5F3DC4,stroke-width:1px,color:#101828
    classDef tst fill:#1b1f24,stroke:#5F3DC4,stroke-width:1px,color:#ffffff
    class CTRL sut
    class S1,S2 dbl
    class TEST tst
```

Each controller is pushed through both halves of its contract: the happy path (correct
`Response`, correct `HttpStatus`, service called exactly once with the expected arguments) and the
full catalogue of failures — `UserNotFound`, `ManagerReservationNotAllowed`,
`ReservationCannotBeCancelled`, `DateTimeInThePast` and friends — each asserted as the
`ResponseException` the API is supposed to surface. The accompanying theory covers test-spy
varieties and when a shared fixture is worth its risks.

**Where to look** → [`CA2/Mizdooni/src/test/java/mizdooni/controllers/`](CA2/Mizdooni/src/test/java/mizdooni/controllers) ·
theory answers in [`CA2/Theory-Questions.md`](CA2/Theory-Questions.md)

---

<a id="ca3"></a>
## ![](https://img.shields.io/badge/CA3-0B7285?style=flat-square) Graph Coverage · JaCoCo

> **The problem.** Turn the Fesadyab fraud engine into a control-flow graph and drive it to maximum
> structural coverage. Statement and branch coverage must both clear 90% — and any node or edge that
> stays unreachable has to be explained, not hidden.

![](https://img.shields.io/badge/control_flow_graph-0B7285?style=flat-square&labelColor=1b1f24)
![](https://img.shields.io/badge/statement_coverage-0B7285?style=flat-square&labelColor=1b1f24)
![](https://img.shields.io/badge/branch_coverage-0B7285?style=flat-square&labelColor=1b1f24)
![](https://img.shields.io/badge/path_feasibility-0B7285?style=flat-square&labelColor=1b1f24)
![](https://img.shields.io/badge/cyclomatic_complexity-0B7285?style=flat-square&labelColor=1b1f24)
![](https://img.shields.io/badge/coverage_instrumentation-0B7285?style=flat-square&labelColor=1b1f24)

`getTransactionPatternAboveThreshold` is the knot of the whole engine — a loop with three guard
clauses and a stateful `diff` accumulator. Drawn as a graph, every test case becomes a visible
walk through it:

```mermaid
flowchart TD
    S(["enter"]) --> E{"history empty?"}
    E -- yes --> R0(["return 0"])
    E -- no --> INIT["diff ← 0<br/>previous ← first txn"]
    INIT --> LOOP{"another txn?"}
    LOOP -- no --> RD(["return diff"])
    LOOP -- yes --> SAME{"txn.id == previous.id?"}
    SAME -- yes --> LOOP
    SAME -- no --> THR{"amount ≤ threshold?"}
    THR -- yes --> LOOP
    THR -- no --> D0{"diff == 0?"}
    D0 -- yes --> SET["diff ← amount − previous.amount<br/>previous ← txn"]
    SET --> LOOP
    D0 -- no --> NEQ{"diff ≠ amount − previous.amount?"}
    NEQ -- yes --> RZ(["return 0 · pattern broken"])
    NEQ -- no --> LOOP

    classDef dec fill:#0B7285,stroke:#063844,stroke-width:0px,color:#ffffff
    classDef act fill:#E2F2F5,stroke:#0B7285,stroke-width:1px,color:#101828
    classDef ret fill:#1b1f24,stroke:#0B7285,stroke-width:1px,color:#ffffff
    class E,LOOP,SAME,THR,D0,NEQ dec
    class INIT,SET act
    class S,R0,RD,RZ ret
```

Reaching every edge means engineering the inputs: an empty history, a duplicate transaction id,
an amount sitting below the threshold, an arithmetic progression that holds, and one that breaks
on the last element. The resulting JaCoCo report leaves nothing uncovered.

<div align="center">

![statements](https://img.shields.io/badge/statement_coverage-100%25-0B7285?style=for-the-badge)
![branches](https://img.shields.io/badge/branch_coverage-100%25-0B7285?style=for-the-badge)
![branches](https://img.shields.io/badge/30%2F30_branches_%C2%B7_45%2F45_lines-4A5568?style=for-the-badge)

</div>

**Where to look** → [`CA3/Fesadyab/src/test/java/domain/`](CA3/Fesadyab/src/test/java/domain) ·
engine under test in [`TransactionEngine.java`](CA3/Fesadyab/src/main/java/domain/TransactionEngine.java)

---

<a id="ca4"></a>
## ![](https://img.shields.io/badge/CA4-087F5B?style=flat-square) Logic-Based Testing, ISP & API Testing

> **The problem.** Test `RestaurantController` and `TableController` *as a web API* — real HTTP
> verbs, real routing, real JSON — never by calling controller methods directly. Then, on paper,
> dissect a predicate with active-clause coverage and partition an input space down to a pair-wise
> test set.

![](https://img.shields.io/badge/API_testing-087F5B?style=flat-square&labelColor=1b1f24)
![](https://img.shields.io/badge/MockMvc-087F5B?style=flat-square&labelColor=1b1f24)
![](https://img.shields.io/badge/@SpringBootTest-087F5B?style=flat-square&labelColor=1b1f24)
![](https://img.shields.io/badge/@MockBean-087F5B?style=flat-square&labelColor=1b1f24)
![](https://img.shields.io/badge/predicate_coverage-087F5B?style=flat-square&labelColor=1b1f24)
![](https://img.shields.io/badge/clause_coverage-087F5B?style=flat-square&labelColor=1b1f24)
![](https://img.shields.io/badge/CACC_%C2%B7_RACC-087F5B?style=flat-square&labelColor=1b1f24)
![](https://img.shields.io/badge/input_space_partitioning-087F5B?style=flat-square&labelColor=1b1f24)
![](https://img.shields.io/badge/pair--wise_coverage-087F5B?style=flat-square&labelColor=1b1f24)

The API tests go through the full Spring stack with the service layer mocked out, so an assertion
is made about the *contract* rather than about a Java return value: status code, JSON body shape,
and the exact error message a client would see.

```java
mockMvc.perform(get("/restaurants/{restaurantId}", restaurant.getId()))
       .andExpect(status().isNotFound())
       .andExpect(jsonPath("$.success").value(false))
       .andExpect(jsonPath("$.message").value("restaurant not found"));
```

The logic half works on `p = (¬a ∧ b) ∨ (b ∧ c) ∨ (¬b ∧ ¬c)`, where the question is not *which
rows* but *which pairs of rows* let a single clause decide the outcome on its own:

| # | `a` | `b` | `c` | `¬a∧b` | `b∧c` | `¬b∧¬c` | **`p`** |
|:--|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
| 1 | T | T | T | F | T | F | **T** |
| 2 | T | T | F | F | F | F | **F** |
| 3 | T | F | T | F | F | F | **F** |
| 4 | T | F | F | F | F | T | **T** |
| 5 | F | T | T | T | T | F | **T** |
| 6 | F | T | F | T | F | F | **T** |
| 7 | F | F | T | F | F | F | **F** |
| 8 | F | F | F | F | F | T | **T** |

<div align="center">

**determining pairs** — `a`: (2,6) · `b`: (1,3) (2,4) (5,7) · `c`: (1,2) (3,4) (7,8)

</div>

Each pair flips exactly one clause while the rest hold still, and `p` flips with it — which is also
why clause coverage alone cannot be trusted to imply predicate coverage here.

**Where to look** → [`CA4/Mizdooni/src/test/java/mizdooni/controllers/`](CA4/Mizdooni/src/test/java/mizdooni/controllers)
(`RestaurantControllerApiTest`, `TableControllerApiTest`) · theory in [`SWT-CA4-TheoryQuestions.pdf`](CA4/SWT-CA4-TheoryQuestions.pdf)

---

<a id="ca5"></a>
## ![](https://img.shields.io/badge/CA5-E8590C?style=flat-square) Mutation Testing & CI Pipeline

> **The problem.** Coverage says the tests *ran* through the code; it never says they would have
> *noticed* if the code were wrong. Mutate the Fesadyab engine with PIT, measure how many mutants
> the suite kills, justify every survivor — and wire the whole thing into a pipeline that re-runs
> on every push.

![](https://img.shields.io/badge/mutation_testing-E8590C?style=flat-square&labelColor=1b1f24)
![](https://img.shields.io/badge/PIT-E8590C?style=flat-square&labelColor=1b1f24)
![](https://img.shields.io/badge/mutation_operators-E8590C?style=flat-square&labelColor=1b1f24)
![](https://img.shields.io/badge/equivalent_mutants-E8590C?style=flat-square&labelColor=1b1f24)
![](https://img.shields.io/badge/test_suite_quality-E8590C?style=flat-square&labelColor=1b1f24)
![](https://img.shields.io/badge/continuous_integration-E8590C?style=flat-square&labelColor=1b1f24)
![](https://img.shields.io/badge/GitHub_Actions-E8590C?style=flat-square&labelColor=1b1f24)
![](https://img.shields.io/badge/refactoring_risk-E8590C?style=flat-square&labelColor=1b1f24)

PIT rewrites the bytecode — negating conditionals, swapping boundaries, discarding return values —
and re-runs the suite against each variant. A mutant that survives is a defect the tests are blind to.

```mermaid
pie showData
    title PIT results · 29 mutants generated
    "Killed by the suite" : 28
    "Survived" : 1
```

The single survivor is an honest one: swapping `>` for `>=` on a comparison that no reachable input
can distinguish — an **equivalent mutant**, undetectable by construction rather than by oversight.
That leaves the suite at **96.5% mutation coverage**, the number that actually justifies refactoring
the engine with confidence.

The pipeline keeps it from rotting:

```mermaid
flowchart LR
    P["push / PR<br/>→ main"] --> CO["checkout"] --> J["set up JDK 21<br/>temurin"] --> CA["cache ~/.m2"] --> T["mvn -B test"] --> JR["jacoco:report"]

    classDef step fill:#FFF0E6,stroke:#E8590C,stroke-width:1px,color:#101828
    classDef trig fill:#E8590C,stroke:#7A2D06,stroke-width:0px,color:#ffffff
    class P trig
    class CO,J,CA,T,JR step
```

**Where to look** → [`.github/workflows/ca5-workflow.yml`](.github/workflows/ca5-workflow.yml) ·
[`CA5/Fesadyab/pom.xml`](CA5/Fesadyab/pom.xml) (PIT + JaCoCo plugins) ·
full write-up in [`CA5-Report.pdf`](CA5/CA5-Report.pdf)

---

<a id="ca6"></a>
## ![](https://img.shields.io/badge/CA6-C2255C?style=flat-square) GUI Testing & Behaviour-Driven Development

> **The problem.** Leave the JVM entirely. Drive Mizdooni through its Swagger UI in a real browser,
> recording a full customer journey with Katalon Recorder and exporting it as runnable
> Selenium WebDriver code — then come back and express domain behaviour as Given/When/Then
> scenarios with Cucumber.

![](https://img.shields.io/badge/GUI_testing-C2255C?style=flat-square&labelColor=1b1f24)
![](https://img.shields.io/badge/record_and_replay-C2255C?style=flat-square&labelColor=1b1f24)
![](https://img.shields.io/badge/Katalon_Recorder-C2255C?style=flat-square&labelColor=1b1f24)
![](https://img.shields.io/badge/Selenium_WebDriver-C2255C?style=flat-square&labelColor=1b1f24)
![](https://img.shields.io/badge/XPath_locators-C2255C?style=flat-square&labelColor=1b1f24)
![](https://img.shields.io/badge/Swagger_UI_%C2%B7_OpenAPI-C2255C?style=flat-square&labelColor=1b1f24)
![](https://img.shields.io/badge/BDD-C2255C?style=flat-square&labelColor=1b1f24)
![](https://img.shields.io/badge/Cucumber_%C2%B7_Gherkin-C2255C?style=flat-square&labelColor=1b1f24)

The recorded scenario is an end-to-end reservation life cycle, clicked through the live API docs:

```mermaid
sequenceDiagram
    autonumber
    actor C as Customer
    participant UI as Swagger UI
    participant API as Mizdooni API

    C->>UI: sign up as a client
    UI->>API: POST /signup
    C->>UI: browse the restaurant list
    UI->>API: GET /restaurants?page=1
    C->>UI: ask for free slots · restaurant 1 · 2 people · 2025-10-26
    UI->>API: GET /reserves/{restaurantId}/available
    API-->>UI: open time slots
    C->>UI: reserve the first open slot
    UI->>API: POST /reserves/{restaurantId}
    C->>UI: review own reservations
    UI->>API: GET /reserves/customer/{customerId}
    C->>UI: cancel the reservation
    UI->>API: POST /reserves/cancel/{reservationNumber}
```

Katalon captures every interaction with a ranked list of candidate locators, and the Java export
turns the suite into a plain JUnit class driving a `FirefoxDriver` — the clearest demonstration of
why XPath-anchored GUI tests are powerful *and* brittle: the resulting selectors are full DOM paths
through Swagger's generated markup.

The BDD half drops back to the domain and re-states its rules as behaviour. Three feature files —
`addReservation`, `addReview` and `getAverageRating` — hold **8 scenarios** written in Gherkin, each
one naming a rule a stakeholder would recognise rather than a method a developer would call:

```gherkin
Scenario: Replacing a review by the same user
  Given a first user
  And a restaurant
  And a review with food 4, service 5, ambiance 4, and overall 4
  When the user adds the review to the restaurant
  And a review with food 2, service 3, ambiance 2, and overall 2
  And the user adds the review to the restaurant
  Then the restaurant should only have 1 review
  And the review in the list should have food 2, service 3, ambiance 2, and overall 2
```

Fifteen step definitions back those sentences with parameterised expressions
(`{int}`, `{double}`), so a scenario reads as prose while still binding to real assertions.
`@CucumberContextConfiguration` boots the Spring context once for the suite, and a JUnit
`TestRunner` glues features to steps and emits an HTML report.

```mermaid
flowchart LR
    F[".feature files<br/>Gherkin scenarios"] --> R["TestRunner<br/>@RunWith(Cucumber.class)"]
    R --> G["StepDefinition<br/>@Given · @When · @Then"]
    G --> D["Mizdooni domain<br/>User · Restaurant"]
    R --> H["target/cucumber-reports<br/>HTML output"]

    classDef gher fill:#C2255C,stroke:#63122F,stroke-width:0px,color:#ffffff
    classDef glue fill:#FCE7EF,stroke:#C2255C,stroke-width:1px,color:#101828
    classDef out  fill:#1b1f24,stroke:#C2255C,stroke-width:1px,color:#ffffff
    class F gher
    class R,G glue
    class D,H out
```

**Where to look** → [`CA6/CA6-TestSuit.krecorder`](CA6/CA6-TestSuit.krecorder) (the recording) ·
[`CA6/CA6-Export.java`](CA6/CA6-Export.java) (the Selenium export) ·
[`features/`](CA6/Mizdooni/src/test/resources/features) (the scenarios) ·
[`StepDefinition.java`](CA6/Mizdooni/src/test/java/mizdooni/stepDefinition/StepDefinition.java) (the glue)

---

## Repository map

```text
SWT-Course-Projects
├── CA1/  Unit Testing ............... Mizdooni · JUnit 5 · model tests + theory
├── CA2/  Test Doubles ............... Mizdooni · Mockito · controller tests + theory
├── CA3/  Graph Coverage ............. Fesadyab · JaCoCo · CFG & branch coverage
├── CA4/  Logic Testing & ISP ........ Mizdooni · MockMvc API tests + CACC/pair-wise theory
├── CA5/  Mutation Testing & CI ...... Fesadyab · PIT · GitHub Actions + report
├── CA6/  GUI Testing & BDD .......... Mizdooni · Katalon/Selenium · Cucumber
└── .github/workflows/
    └── ca5-workflow.yml ............. build · test · coverage on every push to main
```

Inside each assignment folder: the original **`SWTF03-CAx-Description.pdf`** brief, the project
snapshot, and the submitted theory answers.

---

## Running the suites

```bash
# Java 21 is required — verify first
mvn --version          # must report Java 21

# Mizdooni (CA1 · CA2 · CA4 · CA6)
mvn -B test -f CA1/Mizdooni/pom.xml

# Fesadyab + JaCoCo coverage report
mvn -B test -f CA3/Fesadyab/pom.xml
open CA3/Fesadyab/target/site/jacoco/index.html

# Fesadyab + PIT mutation report
mvn -f CA5/Fesadyab/pom.xml org.pitest:pitest-maven:mutationCoverage
open CA5/Fesadyab/target/pit-reports/index.html

# Cucumber BDD scenarios (CA6) — report lands in target/cucumber-reports
mvn -B test -f CA6/Mizdooni/pom.xml -Dtest=TestRunner

# Run Mizdooni and open Swagger UI for the GUI scenarios (CA6)
mvn -f CA6/Mizdooni/pom.xml spring-boot:run
# → http://localhost:8080/api/swagger-ui/index.html
```

---

<div align="center">

### Authors

**Amir Parsa Mobed** · 810100271  
**Arshia Ataei Naeini** · 810100252

<sub>Theory answers are written in Persian. Assignment briefs are the course originals.</sub>

</div>
