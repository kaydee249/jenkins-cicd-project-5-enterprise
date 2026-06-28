# Project 5: Enterprise CI/CD (Automated, Gated, Deployed to Kubernetes)

## Overview

This project brings the whole series together into one pipeline that looks like what a real engineering team runs. It triggers automatically on every push, enforces a set of quality and security gates, and then deploys to a Kubernetes cluster on AWS (EKS). If any gate fails, nothing is pushed or deployed.

It combines two things you have already met: the quality and security gates introduced in this project, and the EKS deployment from Project 4. Read `PRIMER.md` first for the concepts that are new (webhooks and the gates). The Kubernetes concepts come from Project 4, which you should have completed.

This document directs you end to end. You are expected to complete the challenges and submit a written report, described in `REPORT-TEMPLATE.md` and `ASSIGNMENT.md`.

## What you will build

A single pipeline with these stages, in order:

1. Checkout
2. Build
3. Test + Coverage (JaCoCo gate)
4. Code Analysis (SonarQube Quality Gate)
5. Dependency Scan (Trivy)
6. Package
7. Docker Build
8. Image Scan (Trivy)
9. Push to ECR
10. Deploy to EKS

It runs on its own whenever you push, and ends with the new version rolled out across the cluster behind a public URL.

## Prerequisites

- You have completed Project 3 (ECR, the `jenkins-ecr` IAM user and access key) and Project 4 (EKS, `kubectl`, `eksctl`, and how a cluster works).
- An AWS account.
- A server with at least 4 GB RAM, 8 GB recommended, because Jenkins and SonarQube run together. A `t3.large` is a safe choice. The webhook also needs Jenkins reachable from the internet, so this is a cloud project. See `RUN-ON-AWS.md`.
- `PRIMER.md` read.

Cost warning: this project runs an EC2 server, an EKS cluster, worker nodes, and a load balancer. All cost money. Create them when you start and delete them the moment you finish (see Cleanup).

## Architecture

See `architecture.svg` for a one-picture view of the whole system: how a push flows from GitHub through the gates on the build server, to ECR, and out to the EKS cluster.

On the build server, three containers run together (`jenkins/docker-compose.yml`): Jenkins, a SonarQube server, and a Postgres database for SonarQube. Jenkins uses the host's Docker to build, scan, and push images, and it uses `kubectl` to deploy to the separate EKS cluster you create with `eksctl`. The cluster pulls the image straight from ECR, with no pull secret, because its nodes are granted ECR access at creation.

## Practice Kubernetes locally first (free)

If you want to rehearse the Kubernetes parts without paying for EKS, `LOCAL-KUBERNETES.md` runs the same app on a free local cluster (minikube) using `k8s/local/`. Do that to get comfortable, then come back for the pipeline-driven EKS deploy.

## Step 0: Prepare the host

SonarQube needs a kernel setting raised:

```bash
sudo sysctl -w vm.max_map_count=262144
echo 'vm.max_map_count=262144' | sudo tee -a /etc/sysctl.conf
```

## Step 1: Create the EKS cluster

Follow `EKS-SETUP.md`: create the cluster with `eksctl`, map the `jenkins-ecr` user into it, and add the `eks:DescribeCluster` permission. Confirm with `kubectl get nodes`.

## Step 2: Start the build stack

```bash
cd jenkins
docker compose up -d --build
docker compose ps
```

Jenkins is on 8080, SonarQube on 9000. Get the Jenkins password and finish its setup wizard:

```bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

## Step 3: Configure SonarQube

Open `http://<your-host>:9000`, log in with `admin` / `admin`, set a new password. Create a project with key `cicd-project-5`, choose to analyse it Locally, and generate a token. Copy it.

## Step 4: Add Jenkins credentials

Manage Jenkins, Credentials, add:

1. `aws-ecr` (Username and password): username is the AWS Access key ID, password is the secret.
2. `sonar-token` (Secret text): the SonarQube token from Step 3.

## Step 5: Point the pipeline at your account

In the `Jenkinsfile`, set `AWS_REGION`, `ECR_ACCOUNT`, `ECR_REPO`, and `EKS_CLUSTER`. Commit and push.

## Step 6: Create the job and enable the trigger

New Item, name `cicd-project-5`, Pipeline. Under Build Triggers, tick "GitHub hook trigger for GITScm polling". Under Pipeline, choose Pipeline script from SCM, Git, your repo URL, branch `*/main`, Script Path `Jenkinsfile`. Save.

## Step 7: Connect the GitHub webhook

In your GitHub repository, Settings, Webhooks, Add webhook. Payload URL `http://<your-host>:8080/github-webhook/` (trailing slash), content type `application/json`, the push event. If Jenkins is not reachable from GitHub, fall back to the job's Poll SCM option or a tunnel such as ngrok.

## Step 8: Run it end to end

Make a small commit and push. The pipeline starts on its own. Watch all ten stages. On success, get the app URL and open it:

```bash
kubectl get service cicd-app
```

Use the `EXTERNAL-IP` (a load balancer hostname; it takes a couple of minutes the first time). Open `http://<that-hostname>/`, and view your SonarQube dashboard at `http://<your-host>:9000/`.

## Understanding the new stages

- **Test + Coverage** runs the tests and the JaCoCo coverage check. Below 70 percent line coverage fails here. The threshold and exclusions are in `app/pom.xml`.
- **Code Analysis (SonarQube)** runs static analysis and, with `-Dsonar.qualitygate.wait=true`, waits for the Quality Gate and fails if it fails.
- **Dependency Scan (Trivy)** checks your dependencies for known vulnerabilities.
- **Image Scan (Trivy)** checks the built image before it is pushed.
- **Deploy to EKS** points `kubectl` at the cluster, injects this build's image into the Deployment manifest, applies it, and waits for the rollout.

## Challenges

Complete these. Your report must show evidence of each.

1. **Trigger automatically.** Push a commit and show the build started from the webhook (cause: Started by GitHub push).
2. **Fail the coverage gate, then fix it.** Add an untested method to `Calculator` to drop coverage below 70 percent, show the failed stage, then add a test and show it pass.
3. **Fail the Quality Gate.** Introduce a code smell or bug SonarQube flags, show the failed Code Analysis stage and the issue on the dashboard, then fix it.
4. **Catch a vulnerable dependency.** Add a dependency with a known vulnerability, show the Dependency Scan failing with the CVE, then remove it.
5. **Roll out a new version through the pipeline.** Change the app's message, push, and show the webhook-triggered build deploying the new version to EKS, with `kubectl rollout status` succeeding and the new message live at the load balancer URL.

## Troubleshooting

- **SonarQube keeps restarting.** `vm.max_map_count` not applied, or low memory. Re-run Step 0, use a larger instance.
- **Code Analysis fails on the gate.** Check the `sonar-token` credential and that `SONAR_HOST` matches the service name. First analysis takes a minute.
- **Webhook does not fire.** GitHub cannot reach Jenkins. Confirm port 8080 is open to GitHub, the URL ends in `/github-webhook/`, and the trigger is ticked. Use Poll SCM as a fallback.
- **Deploy stage: Unauthorized.** The `jenkins-ecr` user is not mapped into the cluster. Run the `eksctl create iamidentitymapping` command from `EKS-SETUP.md`.
- **Deploy stage: AccessDenied on eks:DescribeCluster.** Add the inline policy from `EKS-SETUP.md`.
- **Pods in ImagePullBackOff.** Confirm the cluster was made with `eksctl` and the account id and region in the image name are right.
- **EXTERNAL-IP stays pending.** Give the load balancer two or three minutes.

## Cleanup (this costs money)

```bash
kubectl delete -f k8s/service.yaml             # remove the load balancer first
eksctl delete cluster --name cicd-cluster --region us-east-1
cd jenkins && docker compose down -v
```

If you ran on EC2, terminate the instance. Delete the ECR images and the IAM access key when finished with the series.

## Deliverable

Complete all the challenges, then write a full report following `REPORT-TEMPLATE.md`. Submit as described in `ASSIGNMENT.md`.
