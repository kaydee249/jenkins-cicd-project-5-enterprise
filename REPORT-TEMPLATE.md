# Project 5 Report Template

Write your report by completing every section below. Replace the guidance in italics with your own work. Keep it professional: clear, specific, and supported by screenshots. Aim for something you would be comfortable showing an employer.

Submit the report as a PDF named `Project5-Report-<YourName>.pdf`.

---

## Cover

- Project: Enterprise CI with Automation and Quality and Security Gates
- Your name:
- Date:
- GitHub repository link:

## 1. Objective

*State, in your own words, what this project set out to achieve and why automated gates matter.*

## 2. Architecture

*Describe the system you built: Jenkins, SonarQube, the database, and the tools used (JaCoCo, Trivy). Explain the pipeline stages and what each does. Include a simple diagram or the stage list.*

## 3. Environment setup

*Describe how you set the environment up: the host, the `vm.max_map_count` step, starting the stack, configuring SonarQube, and adding the Jenkins credentials. Include screenshots of SonarQube running and of the two credentials configured (hide the secret values).*

## 4. The pipeline

*Walk through your `Jenkinsfile`. For each new stage (Test + Coverage, Code Analysis, Dependency Scan, Image Scan) explain what it does and how it acts as a gate, and explain the Deploy to EKS stage. Include a screenshot of a full green run showing all ten stages, and the app open at the load balancer URL.*

## 5. Automation

*Show that the pipeline triggers automatically. Include the GitHub webhook configuration and a build whose cause is "Started by GitHub push".*

## 6. Challenges completed

*For each of the five challenges in the README, write a short subsection: what you did, what happened, and the evidence. Include the before (failure) and after (fixed) screenshots where the challenge asks for them.*

- 6.1 Automatic trigger
- 6.2 Coverage gate failed then fixed
- 6.3 Quality Gate failed then fixed
- 6.4 Vulnerable dependency caught
- 6.5 Rolling update deployed to EKS through the pipeline

## 7. Problems encountered

*Honestly describe anything that went wrong and how you solved it. This section matters; real engineering is mostly solving unexpected problems.*

## 8. What you learned

*Reflect on the new ideas: webhooks, coverage, static analysis, dependency and image scanning, and quality gates. What surprised you? What would you do differently?*

## 9. Conclusion

*Summarise what you delivered and how it differs from the earlier projects.*

## Appendix

- Links to your repository and any references you used.
