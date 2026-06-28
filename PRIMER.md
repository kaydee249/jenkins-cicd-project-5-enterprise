# Primer: New Concepts in Project 5

Read this before you start. Projects 1 to 4 taught you to build, test, package, push, and deploy. This project adds the things that make a pipeline trustworthy in a real company: it runs on its own, and it refuses to ship code that fails quality or security checks. Below are the ideas you have not met yet.

## Automated triggers and webhooks

Until now you clicked Build Now to run the pipeline. In a real team, a build should start by itself the moment someone pushes code. That is done with a webhook.

A webhook is a message one system sends to another when an event happens. Here, GitHub sends a small HTTP request to Jenkins every time you push. Jenkins receives it and starts the pipeline. You write the code, push, and the pipeline runs without you. (The older alternative is polling, where Jenkins repeatedly asks GitHub "anything new?" on a schedule. Webhooks are instant and cheaper, so they are preferred.)

For a webhook to work, GitHub must be able to reach your Jenkins over the internet, so Jenkins needs a public address. That is why this project is best run on a cloud server.

## Code coverage

Passing tests tell you the tests you wrote succeeded. They do not tell you how much of your code those tests actually exercised. Code coverage measures that: the percentage of your code lines that ran during the tests. Low coverage means large parts of your app are untested and could hide bugs.

This project uses JaCoCo, a Java coverage tool, and sets a gate: if line coverage drops below a threshold, the build fails. That pushes you to keep tests meaningful as the code grows.

## Static analysis and SonarQube

Static analysis reads your source code without running it and flags problems: likely bugs, risky patterns, security weaknesses, and "code smells" that make code hard to maintain. It is also called SAST, static application security testing.

SonarQube is a popular static-analysis server. Your pipeline sends the code to SonarQube, which analyses it and applies a Quality Gate: a set of conditions the code must meet, for example no new bugs and enough coverage. If the Quality Gate fails, the pipeline stops. SonarQube also gives you a dashboard to see exactly what it found.

## Dependency scanning (software composition analysis)

Most of your app is code you did not write: open-source libraries you depend on. Some of those libraries have known security vulnerabilities. Dependency scanning, also called SCA, software composition analysis, checks your dependencies against databases of known vulnerabilities and warns you.

This project uses Trivy to scan the project's dependencies. If a serious, fixable vulnerability is found, the build fails so you upgrade before shipping.

## Container image scanning

Your final artifact is a Docker image, which contains not just your app but an operating system and system libraries. Those can have vulnerabilities too. Image scanning inspects the built image for known issues.

This project uses Trivy again, this time on the image, and fails on fixable critical vulnerabilities.

## Quality gates and shifting left

The pattern tying all of this together is the quality gate: an automatic check that must pass before the pipeline continues. Coverage, SonarQube, and the two Trivy scans are all gates. If any fails, nothing gets pushed or deployed.

This is called shifting left: catching problems as early as possible, at the moment of the push, rather than after they reach production where they are far more expensive to fix. A pipeline with gates is the difference between "it built" and "it is safe to ship."

## How these appear in the pipeline

Your Project 5 pipeline has these new stages, in order:

1. Test + Coverage, which now enforces the JaCoCo coverage gate.
2. Code Analysis (SonarQube), which runs static analysis and waits on the Quality Gate.
3. Dependency Scan (Trivy), which checks your dependencies.
4. Image Scan (Trivy), which checks the built image before it is pushed.

Everything else (build, package, push to ECR) you already know from Project 3, and the final Deploy to EKS stage is the Kubernetes deploy from Project 4. What is genuinely new here is the automation and the four gates above.
