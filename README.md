Contribution 1: Bug Report: disabled tests for GCP / VertexAIContribution Number: 1
Student: Luciana Galvez
Issue: https://github.com/traceloop/openllmetry/issues/417
Status: Phase II Complete
Why I Chose This Issue: I chose this issue because OpenLLMetry is directly connected to AI/LLM observability, which fits well with my AI open source capstone. The issue focuses on VertexAI instrumentation tests, so it gives me a chance to learn how real AI tooling projects validate integrations without making live calls to external services.
This issue also looks appropriate for a first open source contribution because it is labeled both good first issue and help wanted. It is scoped around testing rather than redesigning the whole library, but it still requires me to understand the project structure, the instrumentation behavior, and how maintainers expect tests to be written.
Understanding the IssueProblem DescriptionThe OpenLLMetry project has disabled tests for the GCP / VertexAI instrumentation because the previous recording approach, vcr.py, does not support GRPC requests. The maintainers need a way to mock or simulate the VertexAI GRPC interactions so the tests can run without making real network calls.
Expected BehaviorThe VertexAI instrumentation should have automated tests that can run locally and in CI without calling real GCP / VertexAI services. These tests should verify that the instrumentation creates the expected telemetry spans and attributes.
Current BehaviorThe issue says the VertexAI tests were disabled after a related change because the current test-recording approach cannot handle GRPC. As a result, the project has weaker test coverage for VertexAI instrumentation.
Affected ComponentsVertexAI Instrumentation
GCP / VertexAI test suite
Test mocking or fixture setup for GRPC-based requests
OpenTelemetry span/attribute verification for VertexAI behavior
Reproduction ProcessEnvironment SetupLocal setup notes:
Forked traceloop/openllmetry.
Cloned the repository locally.
On Windows, the first clone checkout failed because some test cassette filenames were too long. I fixed the checkout by running git config core.longpaths true inside the repo and then git checkout -f HEAD.
Installed the repo's Node/Nx dependencies with npm ci.
Located the VertexAI instrumentation package at packages/opentelemetry-instrumentation-vertexai.
Confirmed the package test command from project.json: uv run pytest tests/.
Attempted to run npx nx run opentelemetry-instrumentation-vertexai:test; the current local environment stops at uv not being recognized, so the next local setup task is installing uv correctly on PATH.
Important finding: the old VertexAI behavior tests are still present but disabled by filename:
packages/opentelemetry-instrumentation-vertexai/tests/disabled_test_bison.py
packages/opentelemetry-instrumentation-vertexai/tests/disabled_test_gemini.py
Because pytest discovers files named like test_*.py, these disabled_test_*.py files are not collected by the normal package test command.
Steps to ReproduceFork https://github.com/traceloop/openllmetry.
Clone the fork locally:bash



git clone https://github.com/<your-username>/openllmetry.git
cd openllmetry


If Windows fails checkout because of long paths, run:bash



git config core.longpaths true
git checkout -f HEAD


Install the repo-level Node/Nx dependencies:bash



npm ci


Open the VertexAI instrumentation package:bash



cd packages/opentelemetry-instrumentation-vertexai


Inspect the test files:bash



dir tests


Observe that the old VertexAI behavior tests are named disabled_test_bison.py and disabled_test_gemini.py, so they are not collected by the normal pytest pattern.
Return to the repo root and run the package test target:bash



cd ../..
npx nx run opentelemetry-instrumentation-vertexai:test


The package test target runs uv run pytest tests/. With uv properly installed, this command should collect the enabled tests only. In my current Windows setup, the command reaches the target but fails earlier with 'uv' is not recognized as an internal or external command.
Open tests/disabled_test_bison.py and tests/disabled_test_gemini.py. These disabled tests still make real VertexAI SDK calls such as TextGenerationModel.from_pretrained(...).predict(...), ChatModel.from_pretrained(...).start_chat(...), and GenerativeModel(...).generate_content(...). Several Bison tests are marked with pytest.mark.vcr, while the GitHub issue explains that VCR cannot handle the GRPC requests used by VertexAI.
Reproduction EvidenceBranch showing reproduction: https://github.com/YOUR-USERNAME/openllmetry/tree/fix-vertexai-disabled-tests
Screenshots/logs: Local command findings:npm ci completed successfully after allowing npm to write cache/log files.
npx nx run opentelemetry-instrumentation-vertexai:test reached the package target and attempted uv run pytest tests/.
The current local machine needs uv fixed on PATH before pytest can run: 'uv' is not recognized as an internal or external command.

My findings: The reproduction confirms the issue described by maintainers: the real VertexAI behavior tests are present but disabled. The affected files are disabled_test_bison.py and disabled_test_gemini.py, and the tests rely on live VertexAI SDK calls and VCR-style recording even though VCR does not support the GRPC traffic involved.
Solution ApproachAnalysisThe likely root cause is that the existing test approach depends on recording/replaying HTTP interactions, but VertexAI requests may use GRPC. Since vcr.py cannot replay those GRPC calls, tests either make real service calls or have to be disabled. A better test setup should mock the VertexAI client or GRPC layer directly.
Proposed SolutionI plan to investigate how other OpenLLMetry instrumentations test external provider calls, then apply a similar pattern to VertexAI. The proposed fix is to replace the unsupported recording strategy with deterministic mocks or test doubles that allow the instrumentation behavior to be tested without external network access.
Implementation PlanUsing UMPIRE framework (adapted):
Understand: VertexAI instrumentation needs tests that verify telemetry output without depending on live GRPC calls.
Match: I will look for existing tests in the OpenLLMetry repository that mock AI provider SDKs, especially OpenAI or other provider instrumentation tests.
Plan:
Fix local setup by installing uv correctly and confirming npx nx run opentelemetry-instrumentation-vertexai:test runs the enabled tests.
Use disabled_test_gemini.py as the first target because it covers the newer GenerativeModel.generate_content path.
Study the wrapping points in opentelemetry/instrumentation/vertexai/__init__.py for generate_content, send_message, predict, and streaming methods.
Replace live VertexAI calls with deterministic mocks/fakes for model responses, usage metadata, candidates, and response text.
Rename or split the disabled tests into normal test_*.py files once they no longer require real GCP credentials, real network calls, or VCR.
Re-run the VertexAI package test target and verify spans/logs still assert the expected model, prompt, completion, token, and event attributes.
Open a pull request explaining that the disabled tests were restored by mocking the SDK/GRPC boundary instead of replaying GRPC traffic with VCR.
Implement: [Link to my branch/commits as I work - to be added]
Review: I will check that the change follows the project contribution guide, keeps tests deterministic, and avoids real GCP credentials or network calls.
Evaluate: I will verify the fix by running the VertexAI test suite locally and confirming the tests pass without external service access.
Testing StrategyUnit Tests
Test case 1: VertexAI instrumentation creates expected spans for a mocked request.

Test case 2: Span attributes match the project's existing semantic convention expectations.

Test case 3: Tests run without GCP credentials or live GRPC calls.
Integration Tests
Integration scenario 1: Run the VertexAI instrumentation test group locally.

Integration scenario 2: Confirm the relevant package test command passes in the fork.
Manual TestingManual testing will include running the affected tests before and after the fix, then documenting the observed results and any setup issues.
Implementation NotesWeek 1 ProgressSelected the issue, reviewed the issue description, confirmed that it is open and labeled good first issue and help wanted, and drafted the Phase I contribution README.
Week 2 ProgressCompleted Phase II reproduction work. I cloned the project, resolved a Windows long-path checkout issue, installed the repo-level Nx dependencies, located the VertexAI package, and confirmed that the old behavior tests are disabled by filename. I also identified the package test command and found that my current machine still needs uv correctly installed on PATH before pytest can run locally.
Code ChangesFiles modified: [To be added after implementation]
Key commits: [To be added after implementation]
Approach decisions: [To be added after implementation]
Pull RequestPR Link: [GitHub PR URL when submitted]
PR Description: [Draft or final PR description - much of the content above can be adapted]
Maintainer Feedback:
[Date]: [Summary of feedback received]
[Date]: [How I addressed it]
Status: Not submitted yet
Learnings & ReflectionsTechnical Skills Gained[To be updated as I work on the issue]
Challenges Overcome[To be updated as I work on the issue]
What I'd Do Differently Next Time[To be updated after completing the contribution]
Resources UsedSelected issue: https://github.com/traceloop/openllmetry/issues/417
OpenLLMetry repository: https://github.com/traceloop/openllmetry
OpenLLMetry contribution page: https://github.com/traceloop/openllmetry/contribute
Original GitHub issue search: https://github.com/issues?q=is%3Aopen+is%3Aissue+label%3A%22help+wanted%22
