# Running the Project on AWS (EC2)

This runs the exact same Dockerized Jenkins on a cloud server instead of your laptop. Once it is up, you reach Jenkins from your browser at the server's public address. These steps work for either project, Python or Java.

Cost warning: an EC2 instance costs money while it runs. Use a small instance and terminate it when you are done. Consider setting a billing alert in the AWS Console first.

## 1. Launch an EC2 instance

1. Sign in to the AWS Console, open EC2, and click Launch instance.
2. Name it, for example `jenkins-cicd`.
3. AMI: choose Ubuntu Server 24.04 LTS (or Amazon Linux 2023).
4. Instance type: choose `t3.large` (2 vCPU, 8 GB RAM). Project 5 runs Jenkins, SonarQube, and a database together, plus builds, so it needs more memory than the earlier projects. Smaller instances will make SonarQube fail to start.
5. Key pair: create or select one so you can SSH in, and download the `.pem` file.
6. Network settings, Edit the security group inbound rules to allow:
   - SSH, port 22, source My IP.
   - Custom TCP, port 8080. For Project 5 the GitHub webhook must reach Jenkins, so this port has to be open to GitHub. Either allow `0.0.0.0/0` here (acceptable for a short-lived learning server) or restrict it to GitHub's published webhook IP ranges. For Jenkins login from your browser, your own IP is also covered by this rule.
   - Custom TCP, port 9000, source My IP, so you can reach the SonarQube dashboard.
7. Storage: 20 GB or more.
8. Launch instance.

## 2. Connect with SSH

From your machine (Linux or macOS terminal, or Windows PowerShell):

```bash
chmod 400 your-key.pem                          # Linux and macOS only
ssh -i your-key.pem ubuntu@<EC2-PUBLIC-IP>      # Ubuntu AMI
# For Amazon Linux, the user is ec2-user:
# ssh -i your-key.pem ec2-user@<EC2-PUBLIC-IP>
```

Replace `<EC2-PUBLIC-IP>` with the Public IPv4 address shown on the instance page. On Windows, the same `ssh` command works in PowerShell, or you can use PuTTY.

## 3. Install Docker and Git on the server

Ubuntu:

```bash
sudo apt update
sudo apt install -y docker.io docker-compose-v2 git
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
newgrp docker          # apply the docker group without logging out
```

Amazon Linux 2023:

```bash
sudo dnf install -y docker git
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
newgrp docker
# install the docker compose plugin
sudo mkdir -p /usr/libexec/docker/cli-plugins
sudo curl -SL https://github.com/docker/compose/releases/latest/download/docker-compose-linux-x86_64 \
  -o /usr/libexec/docker/cli-plugins/docker-compose
sudo chmod +x /usr/libexec/docker/cli-plugins/docker-compose
```

Verify:

```bash
docker --version
docker compose version
```

## 4. Get the project onto the server

Option A, clone from your GitHub repo (after you have pushed it):

```bash
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>
```

Option B, copy from your own machine with `scp` (run this on your machine, not on the server):

```bash
scp -i your-key.pem -r ./jenkins-cicd-project-5-enterprise ubuntu@<EC2-PUBLIC-IP>:~/
```

## 5. Start Jenkins

```bash
cd jenkins
docker compose up -d --build
```

Get the one-time unlock password:

```bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

## 6. Open Jenkins in your browser

Go to:

```
http://<EC2-PUBLIC-IP>:8080
```

Paste the unlock password, install suggested plugins, and create your admin user. Then create the pipeline job exactly as in the project README: Pipeline script from SCM, your repository URL, branch `*/main`.

If the page does not load, the security group is not allowing port 8080 from your current IP, or your IP has changed since you set the rule. Edit the inbound rules and try again.

## 7. (Optional) Install Maven on the server

You do not need Maven on the server for the pipeline, because the Jenkins image already has it. Install it only if you want to run `mvn` directly on the box. Follow the Linux section of `INSTALL-MAVEN.md`.

## 8. Clean up to stop charges

```bash
cd jenkins
docker compose down -v        # stop Jenkins and remove its data
```

Then, in the EC2 Console, either Stop the instance (keeps it for later, small storage charge) or Terminate it (deletes it, no further charge). Terminate when you are completely finished.

## Local vs AWS: what actually changes

Nothing in the project changes. The compose file, the `Jenkinsfile`, and the pipeline are identical. The only difference is the address you use to reach Jenkins:

- Locally: `http://localhost:8080`
- On AWS: `http://<EC2-PUBLIC-IP>:8080`

That is the whole point. A pipeline you build on your laptop runs the same way on a server.

## Project 5 note: ports and where the app runs

This server runs Jenkins and SonarQube. The app itself runs on your EKS cluster, not on this server.

- Jenkins at `http://<EC2-PUBLIC-IP>:8080/` (also the webhook target at `/github-webhook/`)
- SonarQube at `http://<EC2-PUBLIC-IP>:9000/`
- The app at the EKS load balancer URL from `kubectl get service cicd-app` (the EXTERNAL-IP)

Run the host preparation step before starting the stack: `sudo sysctl -w vm.max_map_count=262144`, or SonarQube will not start.
