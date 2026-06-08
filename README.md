# su26-ai301-contribution
Contribution 1: Bug Report: disabled tests for GCP / VertexAI
Contribution Number: 1
Student: Luciana Galvez
Issue: https://github.com/traceloop/openllmetry/issues/417
Status: Phase I Complete

Why I Chose This Issue
I chose this issue because OpenLLMetry is directly connected to AI/LLM observability, which fits well with my AI open source capstone. The issue focuses on VertexAI instrumentation tests, so it gives me a chance to learn how real AI tooling projects validate integrations without making live calls to external services.

This issue also looks appropriate for a first open source contribution because it is labeled both good first issue and help wanted. It is scoped around testing rather than redesigning the whole library, but it still requires me to understand the project structure, the instrumentation behavior, and how maintainers expect tests to be written.

Understanding the Issue
Problem Description
The OpenLLMetry project has disabled tests for the GCP / VertexAI instrumentation because the previous recording approach, vcr.py, does not support GRPC requests. The maintainers need a way to mock or simulate the VertexAI GRPC interactions so the tests can run without making real network calls.

Expected Behavior
The VertexAI instrumentation should have automated tests that can run locally and in CI without calling real GCP / VertexAI services. These tests should verify that the instrumentation creates the expected telemetry spans and attributes.

Current Behavior
The issue says the VertexAI tests were disabled after a related change because the current test-recording approach cannot handle GRPC. As a result, the project has weaker test coverage for VertexAI instrumentation.

Affected Components
VertexAI Instrumentation
GCP / VertexAI test suite
Test mocking or fixture setup for GRPC-based requests
OpenTelemetry span/attribute verification for VertexAI behavior

Reproduction Process
Environment Setup
Planned setup:

Fork traceloop/openllmetry.
Clone my fork locally.
Read the repository CONTRIBUTING.md.
Install the Python development dependencies required for the VertexAI instrumentation package.
Locate the disabled VertexAI tests and run the related test target.
Current Phase I status: I have selected the issue and documented my initial understanding. Full local reproduction will be completed in the next phase.

Steps to Reproduce
Clone and set up the OpenLLMetry repository.
Navigate to the package or test area for VertexAI instrumentation.
Run the VertexAI-related tests.
Observe that the relevant tests are disabled or cannot be run with the existing vcr.py approach because VertexAI uses GRPC.
Reproduction Evidence
Commit showing reproduction: [Link to commit in my fork - to be added in Phase II]

My findings: The current issue report says tests were disabled because the previous request-recording strategy does not support GRPC.

Solution Approach
Analysis
The likely root cause is that the existing test approach depends on recording/replaying HTTP interactions, but VertexAI requests may use GRPC. Since vcr.py cannot replay those GRPC calls, tests either make real service calls or have to be disabled. A better test setup should mock the VertexAI client or GRPC layer directly.

Proposed Solution
I plan to investigate how other OpenLLMetry instrumentations test external provider calls, then apply a similar pattern to VertexAI. The proposed fix is to replace the unsupported recording strategy with deterministic mocks or test doubles that allow the instrumentation behavior to be tested without external network access.

Implementation Plan


Understand: VertexAI instrumentation needs tests that verify telemetry output without depending on live GRPC calls.

Match: I will look for existing tests in the OpenLLMetry repository that mock AI provider SDKs, especially OpenAI or other provider instrumentation tests.

Plan:

Identify the disabled VertexAI tests and the code paths they were meant to cover.
Find the instrumentation entry points and expected span attributes.
Replace live/recorded VertexAI calls with mocks, fixtures, or fake client responses.
Re-enable the relevant tests.
Run the package-specific test suite and update the README with reproduction evidence.
Implement: [Link to my branch/commits as I work - to be added]

Review: I will check that the change follows the project contribution guide, keeps tests deterministic, and avoids real GCP credentials or network calls.

Evaluate: I will verify the fix by running the VertexAI test suite locally and confirming the tests pass without external service access.

Testing Strategy
Unit Tests
Test case 1: VertexAI instrumentation creates expected spans for a mocked request.
Test case 2: Span attributes match the project's existing semantic convention expectations.
Test case 3: Tests run without GCP credentials or live GRPC calls.
Integration Tests
Integration scenario 1: Run the VertexAI instrumentation test group locally.
Integration scenario 2: Confirm the relevant package test command passes in the fork.
Manual Testing
Manual testing will include running the affected tests before and after the fix, then documenting the observed results and any setup issues.

Implementation Notes
Week 1 Progress
Selected the issue, reviewed the issue description, confirmed that it is open and labeled good first issue and help wanted, and drafted the Phase I contribution README.

Week 2 Progress
[To be updated after fork, setup, and reproduction]

Code Changes
Files modified: [To be added after implementation]
Key commits: [To be added after implementation]
Approach decisions: [To be added after implementation]

Pull Request
PR Link: [GitHub PR URL when submitted]

PR Description: [Draft or final PR description - much of the content above can be adapted]

Maintainer Feedback:

[Date]: [Summary of feedback received]
[Date]: [How I addressed it]
Status: Not submitted yet

Learnings & Reflections
Technical Skills Gained
[To be updated as I work on the issue]

Challenges Overcome
[To be updated as I work on the issue]

What I'd Do Differently Next Time
[To be updated after completing the contribution]

Resources Used
Selected issue: https://github.com/traceloop/openllmetry/issues/417
OpenLLMetry repository: https://github.com/traceloop/openllmetry
OpenLLMetry contribution page: https://github.com/traceloop/openllmetry/contribute
Original GitHub issue search: https://github.com/issues?q=is%3Aopen+is%3Aissue+label%3A%22help+wanted%22
