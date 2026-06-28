# Project 5 Assignment: Enterprise CI/CD to Kubernetes

**Due: [set your deadline]**

Two deliverables: a working automated, gated pipeline that deploys to EKS, and a professional written report. Read `PRIMER.md`, follow `README.md` end to end, complete all five challenges, and document everything in your report.

## What to do

1. Create the EKS cluster and grant the Jenkins user access (`EKS-SETUP.md`).
2. Stand up the build stack (Jenkins, SonarQube, database) and configure SonarQube and the Jenkins credentials.
3. Build the ten-stage pipeline and get one fully green run that deploys to EKS.
4. Set up the GitHub webhook so builds trigger automatically.
5. Complete all five challenges in the README.
6. Write your report using `REPORT-TEMPLATE.md`.

## What to submit

1. The link to your GitHub repository.
2. A screenshot of a green pipeline run showing all ten stages, including the four gates and Deploy to EKS.
3. A screenshot of a build triggered by the GitHub webhook (cause: Started by GitHub push).
4. A screenshot of your project in the SonarQube dashboard with its Quality Gate result.
5. A screenshot of `kubectl get service cicd-app` showing the load balancer URL, and the app open in a browser at that URL.
6. A screenshot proving the rolling update from Challenge 5 (the new message live after a pipeline deploy).
7. Your completed report as a PDF (`Project5-Report-<YourName>.pdf`), covering all sections and all five challenges.

## Grading (100 points)

- Working ten-stage pipeline with one fully green run that deploys to EKS: 25
- Automatic trigger via webhook demonstrated: 10
- Four gates shown working (coverage, SonarQube, dependency, image): 15
- All five challenges completed with evidence: 25
- Report: complete, clear, professional, follows the template: 25

## Reminder

This project costs real money: an EC2 server, an EKS cluster, nodes, and a load balancer. Delete the Service first, then the cluster (`eksctl delete cluster`), stop the stack, terminate the instance, and delete the ECR images and IAM access key when done.

## How to submit

Reply to the assignment email with your repository link, the screenshots, and your report PDF attached.
