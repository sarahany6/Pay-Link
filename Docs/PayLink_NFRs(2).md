# PayLink: Non-Functional Requirements


## Part 1: Overview Table

| NFR ID | Quality Attribute | Short Description | Priority |
|---|---|---|---|
| NFR-01 | Correctness | Zero double charges and zero negative balances in the concurrency and duplicate tests | Must |
| NFR-02 | Performance | 100 payments per second sustained for 1 minute against the mock billers | Should |
| NFR-03 | Resilience | A biller outage does not stop payments to other billers | Must |
| NFR-04 | Modifiability | Adding a biller touches only the Biller Integration module and configuration | Must |
| NFR-05 | Auditability | 100% of money movements have an audit record with who, when and why | Must |
| NFR-06 | Testability | At least 70% line coverage on domain logic, measured in CI | Must |

---

## Part 2: Detailed Requirements

### NFR-01: Payment Correctness
- **Quality attribute:** Correctness
- **Requirement:** The system shall produce zero double charges and zero negative wallet balances in the concurrency and duplicate tests. A payment that is Rejected shall not create a charge ledger entry.
- **Target / measurement:**
  - Double charges: 0.
  - Negative balances: 0.
  - Measured separately for successful and rejected payments:

    | Payment outcome | Expected charge ledger entries | Expected wallet effect |
    |---|---|---|
    | Successful (Completed) | Exactly 1 per payment | Balance reduced once by the total (amount + fee) |
    | Rejected | 0 | No change to the balance, and no hold remains |

  - Minimum wallet balance observed must be >= 0.00 EGP.
  - Test reports shall give the count of successful payments, the count of rejected payments, the charge entries for each group, and the minimum balance observed.
- **Verification method:**
  - Automated test for hard case 1: the same request with the same idempotency key is sent twice, and the test asserts one payment and one charge. The second request returns the first result and creates no new ledger entry.
  - Automated test for hard case 4: two simultaneous payments of 352.50 EGP against a 400.00 EGP balance, and the test asserts that one succeeds (1 charge entry), one is Rejected (0 charge entries), and the balance is never negative.
  - Test reports must show the measured numbers for successful and rejected payments separately.
- **Source:** Student Brief §3.3 (Correctness); supporting context in §2.5 (Rejected is a final state reached before funds are held), §2.7 (hard cases 1 and 4), §3.4 (Tests), FR-9, FR-10 and FR-12.
- **Priority:** Must. The brief says PayLink holds customers' money and that double charges cause loss of trust and money.

### NFR-02: Payment Throughput
- **Quality attribute:** Performance
- **Requirement:** The system shall sustain 100 payments per second for 1 minute against the mock billers. Requests received and payments processed shall be measured and reported separately, and a received request shall not be counted as a successfully processed payment.
- **Target / measurement:** 100 payments per second, sustained for 1 minute (about 6,000 payments in total). The load test report shall give these figures separately:
  - Requests received (total and per second).
  - Payments processed, split by outcome:
    - Successful (Completed).
    - Rejected (for example insufficient funds or a fraud rule).
    - Failed (for example refused by the biller).
  - Payments still pending at the end of the test (for example Unknown, or SentToBiller), if any.
  - Only successful payments count as successfully processed. Rejected and Failed payments are reported but not counted as successes.
- **Verification method:** A load test (for example k6, Locust or JMeter) that drives payment requests at the mock billers for 60 seconds. The report shows the requests received per second, the achieved rate of processed payments, and the counts of successful, rejected, failed and pending payments. The team shall state the test conditions used (see Ambiguities A1 to A4).
- **Source:** Student Brief §3.3 (Performance); context in §1 ("ready to handle the busiest day of the month") and §2.5 (payment states).
- **Priority:** Should. The brief lists this target among the expected ones and says tests or reports must show the number, so measuring and reporting it is required. It is "Should" rather than "Must" because failing to reach the rate does not by itself lose or corrupt money (that is covered by the Must requirements NFR-01 and NFR-03), and the brief leaves the test conditions open (hardware, biller mix, number of wallets; see A1 to A4). The team should confirm this priority with the instructor or TA.

### NFR-03: Biller Outage Isolation
- **Quality attribute:** Resilience
- **Requirement:** The system shall continue to process payments to all other billers when one biller is in outage. An outage in one biller shall not block payment processing for healthy billers.
- **Target / measurement:** Payments to healthy billers continue to be processed while one mock biller is forced to fail or time out. During the outage test, the report shall give, for the healthy billers:
  - Success rate (successful payments divided by payments submitted).
  - Response time (for example average and 95th percentile).
  - The same two figures for the same billers without an outage, for comparison.
  - The brief states no specific success-rate or response-time target for this test, so none is claimed here. Any threshold shall be agreed by the team and confirmed with the instructor or TA (see A6).
- **Verification method:**
  - Integration test in which one mock biller (for example ElectroCo) is configured to fail or hang, while payments to AquaCo and TopUpNet are submitted and must complete normally.
  - Payments to the failed biller must follow the Unknown or Failed life cycle, and the other billers' payments must not be blocked by them.
  - The test records the success rate and response time of the healthy billers and includes them in the report.
- **Source:** Student Brief §3.3 (Resilience); context in §2.6 (10-second biller timeout), §2.7 (hard case 2) and §2.4 (mock servers "can be told to fail on purpose").
- **Priority:** Must. One slow biller must not take down payments for the whole platform.

### NFR-04: Biller Extensibility
- **Quality attribute:** Modifiability
- **Requirement:** The system shall allow a new biller to be added by changing only the Biller Integration module and configuration.
- **Target / measurement:** Number of files changed outside the Biller Integration module and configuration = 0.
- **Verification method:**
  - Add a new mock biller (hard case 8) and show the version-control diff, where every changed or added file is in the Biller Integration module or in configuration.
  - Repeat the check for the ElectroCo API v2 change in Phase 2, listing the changed files.
- **Source:** Student Brief §3.3 (Modifiability); context in §2.4 (ElectroCo v2 change request), §2.7 (hard case 8) and §4 (Phase 2 item 5).
- **Priority:** Must. Management needs "a system that is easy to grow," and the ElectroCo v2 change is a graded item.

### NFR-05: Money-Movement Audit Trail
- **Quality attribute:** Auditability
- **Requirement:** The system shall record an audit record containing who, when and why for 100% of money movements.
- **Target / measurement:** 100% coverage. Measure it as the number of money movements (ledger entries) that have a matching audit record, divided by the total number of money movements.
- **Verification method:**
  - Automated test that performs every money-moving operation (top-up, hold, charge, release, refund, commission) and asserts that each ledger entry has an audit record with non-empty who, when and why fields.
  - A database query comparing ledger-entry counts with audit-record counts.
- **Source:** Student Brief §3.3 (Auditability); context in FR-5 (ledger) and FR-25 (audit log).
- **Priority:** Must. Regulators need proof that no money is lost.

### NFR-06: Domain Test Coverage
- **Quality attribute:** Testability
- **Requirement:** The system shall achieve at least 70% line coverage on domain logic, measured in the CI pipeline.
- **Target / measurement:** Line coverage on domain logic >= 70%, reported by the CI workflow on every push.
- **Verification method:** A coverage tool (for example JaCoCo, coverage.py or Istanbul) runs in the GitHub Actions workflow. The CI report shows the percentage. Optionally, the build fails below 70%.
- **Source:** Student Brief §3.3 (Testability); context in §3.4 (CI/CD: "reports coverage") and §4 (Phase 2 item 2).
- **Priority:** Must. It is a graded criterion (TDD evidence and coverage above 70%).

---

## Part 3: Ambiguities

| # | Related NFR | Unclear detail | Why it needs clarification | Question to ask the instructor / TA |
|----|--------|-------------------------------------------------------------------|-------------------------------------------------|----------------------------|

| A1 | NFR-02 | Hardware and environment for the load test are not specified. | Results on a laptop and on a server differ greatly. | On what hardware (CPU, RAM, database setup) must the 100 payments per second be shown? |

| A2 | NFR-02 | "Payments" is not defined for the test, including the mix of billers and outcomes. | AquaCo takes 2 to 4 seconds and TopUpNet is asynchronous, so the mix changes the load. | Which billers and which mix should the load test use? Does a payment count when it is accepted, or when it reaches a final state? |

| A3 | NFR-02 | No latency limit (for example a 95th percentile) or error-rate limit is given. | A system can reach the rate while failing or responding very slowly. | Is there a maximum response time or error rate during the test? |

| A4 | NFR-02 | It is not stated how many wallets are used in the test. | Concurrency on one wallet is very different from load spread over many wallets. | Should the load come from many wallets or from a few? |

| A5 | NFR-01 | The zero-defect target applies "in the concurrency and duplicate tests" only. | It is unclear whether this also applies to production-like load, or to failure scenarios such as retries or callbacks. | Should zero double charges also be shown under the load test and the TopUpNet duplicate-callback case? How many concurrent requests must the test use? |

| A6 | NFR-03 | "Outage" is not defined (timeout, error response, or connection refused). | The test setup and expected behaviour differ for each type. | Which outage types must be tested? Is there a limit on how much healthy-biller performance may drop? |

| A7 | NFR-03 | Recovery time and the behaviour after the biller returns are not stated. | The brief covers Unknown retries (1, 5 and 15 minutes) but gives no recovery target. | Is any recovery-time target expected? |

| A8 | NFR-04 | It is not defined whether the Biller Catalog (admin-managed data) counts as "configuration" or as a separate module. | A new biller needs a catalog entry, which could be a data change or a code change. | Does adding a biller record through the admin API count as configuration? Are database migrations allowed? |

| A9 | NFR-04 | The measure of "touches" is not defined (files, packages, classes, or lines). | The result depends on how it is counted. | Is a file-level diff showing no changes outside the integration module and configuration enough? |

| A10 | NFR-05 | "Money movement" is not defined (for example, whether holds and releases count, or only charges and refunds). | The 100% target depends on the scope. | Which operations count as money movements? |

| A11 | NFR-05 | The content of "who" and "why" is not defined for system-initiated actions (Scheduler, callbacks, retries). | A system actor has no user identity. | What should "who" be for automated actions? What is an acceptable "why" (for example a reason code)? |

| A12 | NFR-05 | It is unclear whether admin actions are covered by the 100% target. | FR-25 includes admin actions, but §3.3 mentions money movements only. | Should NFR-05 also cover admin actions? |

| A13 | NFR-06 | "Domain logic" is not defined. | Coverage depends on which packages are counted (for example, whether controllers and adapters are excluded). | Which packages count as domain logic? |

| A14 | NFR-06 | The brief does not say whether the build must fail below 70% or only report it. | It affects the CI configuration. | Must the pipeline enforce the threshold, or only report it? |

---

## Part 4: Assumptions
| A1 | The performance test will be conducted in the team's Docker Compose environment. The hardware specifications, database configuration, and mock biller setup will be documented with the test results |

| A2 |The load test will report both accepted payment requests per second and completed payments per second. Each biller will be tested separately, followed by a mixed-biller test using a documented workload distribution|

| A3 | The performance report will include average latency, p95 latency, and error rate. Until a specific threshold is confirmed, these metrics will be reported without introducing additional mandatory performance limits|

| A4 | The performance test will include both distributed-wallet and shared-wallet scenarios. The number of wallets and the workload distribution will be documented for each scenario|

| A5 | Zero double charges and zero negative wallet balances will be treated as correctness invariants across all applicable tests, including load, duplicate-request, callback, and retry scenarios. The test concurrency level will be documented. |

| A6 | Resilience tests will include timeout, server-error, and connection-failure scenarios. Payments to healthy billers must continue during an outage of another biller. Any performance degradation will be measured and reported |

| A7 | After a biller becomes available, the system will resume processing new payments and continue handling Unknown payments according to the specified retry schedule. Recovery time will be measured and reported |

| A8 | Adding a new biller requires a new integration adapter and a catalog record created through the existing Admin API. No existing module code needs to change | 

| A9 | We will evaluate the modifiability requirement using a file-level diff. New or modified files must be limited to the Biller Integration module and configuration. Any database migration will be documented separately |

| A10 | All wallet money movements, including top-ups, holds, releases, refunds, payments, and agent commissions, will have an audit record containing who initiated the movement, when it occurred, and why |

| A11 | Automated actions will be attributed to the system and the responsible component. The audit record will include a reason code, timestamp, and payment reference. User-initiated actions will include the user or admin identity |

| A12 | The system will audit 100% of money movements and 100% of admin actions, consistent with FR-25. The audit records will identify the actor, timestamp, and reason for each action |

| A13 | Domain logic will include the code responsible for business rules, such as payment state transitions, wallet rules, fee calculations, commissions, and fraud checks. Controllers, DTOs, and infrastructure code will be excluded from the coverage calculation |

| A14 | The CI pipeline will enforce the 70% domain-logic line coverage threshold. The build will fail if coverage falls below the required threshold |


