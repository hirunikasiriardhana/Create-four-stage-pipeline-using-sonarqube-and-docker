# 🔍 SonarQube + Jenkins CI/CD Pipeline

A step-by-step guide to connect **SonarQube** with **Jenkins** and run automated code quality analysis on every build — no prior DevOps wizardry required. ☕🔧

---

## 📦 What I'll Build

```
GitHub Repo ──▶ Jenkins Pipeline ──▶ Maven Build ──▶ SonarQube Analysis ──▶ 📊 Quality Report
```

By the end of this guide, every push to the repo can be automatically checked for bugs, vulnerabilities, and code smells — with zero manual effort.

---

## 🧰 Prerequisites

- SonarQube installed locally (e.g. `sonarqube-26.7.0`)
- Jenkins installed and running
- Maven & JDK available on the machine
- A GitHub repository with project code

---

## 🚀 Step 0: Start SonarQube

Open **Command Prompt as Administrator**, then run:

```cmd
cd C:\sonarqube-26.7.0.124771\bin\windows-x86-64(sonar installed location)
StartSonar.bat
```

Once it's running, open the browser and go to:

👉 `http://localhost:9000`

Log in with SonarQube credentials (default is `admin` / `admin` on first run — change it immediately if you haven't).

---

## Part 1 — Connect SonarQube to Jenkins

### Step 1: Install the SonarQube Scanner Plugin
`Manage Jenkins → Plugins → Available` → search **SonarQube Scanner** → Install → restart Jenkins if prompted.

### Step 2: Generate a Token in SonarQube
`My Account → Security → Generate Token`
- Give it a name, e.g. `jenkins-token`
- Click **Generate**
- ⚠️ **Copy it immediately** — SonarQube only shows it once!

### Step 3: Save the Token as a Jenkins Credential
`Manage Jenkins → Credentials → System → Global credentials → Add Credentials`

| Field | Value |
|---|---|
| Kind | Secret text |
| Secret | *(you can generate token like this sqa_6760113545e44dd7bc4390e393ee236aff647978)* |
| ID | `sonar-token` |

> 💡 Remember this ID — you'll reference it in your pipeline script.

### Step 4: Connect Jenkins to Your SonarQube Server
`Manage Jenkins → System → SonarQube servers`
- ✅ Check **"Environment variables"**
- Click **Add SonarQube**

| Field | Value |
|---|---|
| Name | `SonarQube` |
| Server URL | `http://localhost:9000` |
| Server authentication token | Select the credential from Step 3 |

### Step 5: Configure the Scanner Tool
`Manage Jenkins → Tools → SonarQube Scanner installations → Add`

| Field | Value |
|---|---|
| Name | `SonarScanner` |
| Install automatically | ✅ (or point to a manual path) |

---

## Part 2 — Build the Pipeline

### Step 1: Create a Project in SonarQube
1. Log in to the SonarQube dashboard
2. **Projects → Create Project → Manually**
3. Enter a Project Key (e.g. `sonarpipeline`)
4. Enter a Display Name
5. Click **Set Up**

### Step 2: Set Up JDK & Maven in Jenkins
`Manage Jenkins → Tools`

| Tool | Name | Setup |
|---|---|---|
| JDK | `java` | Add path or enable auto-install |
| Maven | `maven` | Enable auto-install |

> These names must exactly match what's used in the `tools {}` block of the pipeline script below.

### Step 3: Create the Pipeline Job
1. Jenkins Dashboard → **New Item**
2. Name it, e.g. `sonarpipeline`
3. Select **Pipeline** → **OK**
4. Scroll to the **Pipeline** section → Definition: **Pipeline script**
5. Paste the script below 👇

```groovy
pipeline {
    agent any

    tools {
        jdk 'java'
        maven 'maven'
    }

    stages {
        stage('Pull') {
            steps {
                echo 'Pulling source code...'
                checkout scmGit(
                    branches: [[name: '*/master']],
                    extensions: [],
                    userRemoteConfigs: [[url: 'https://github.com/<your-username>/<your-repo>']]
                )
            }
        }

        stage('Build') {
            steps {
                echo 'Building project...'
                bat "mvn clean install"
            }
        }

        stage('Push') {
            steps {
                echo 'Running SonarQube Scanner and pushing analysis...'
                withSonarQubeEnv('SonarQube') {
                    bat "mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.10.0.2594:sonar"
                }
            }
        }
    }
}
```

> Replace `<your-username>/<your-repo>` with your actual GitHub repo URL, and make sure `withSonarQubeEnv('...')` matches the **Name** you set in Part 1, Step 4 (not the credential ID).

6. Click **Save**

### Step 4: Run It! 🎬
1. Open your `sonarpipeline` job
2. Click **Build Now**
3. Watch the **Console Output** — you should see all three stages pass:
   - ✅ Pull
   - ✅ Build
   - ✅ Push (SonarQube Analysis)

### Step 5: Check Your Results
Head back to your SonarQube dashboard and open your project — you'll see a full breakdown of bugs 🐛, vulnerabilities 🔓, and code smells 👃 detected in your codebase.

---

## 🔁 Maintenance Note

SonarQube tokens can expire depending on how they were created. If your pipeline suddenly fails with an **authentication error**:
1. Go back to **Part 1, Step 2** and generate a fresh token
2. Update the Jenkins credential's secret value (Part 1, Step 3) — no need to recreate the credential, just edit the value

---

## 🔒 Security Reminder

**Never commit real tokens, passwords, or API keys to this repository** — not even in commit history. Use:
- Jenkins Credentials Manager for secrets (as shown above)
- A `.gitignore`'d `.env` file for local secrets
- GitHub's [Secret Scanning](https://docs.github.com/en/code-security/secret-scanning) to catch accidental leaks

If a real secret is ever pushed by mistake, **revoke/regenerate it immediately** — deleting the commit is not enough, since it may still exist in Git history or cached forks.

---

## 🎉 That's It!

Now you have a self-checking pipeline: every build pulls your code, compiles it, and grades its code quality automatically. Happy shipping! 🚢

<img width="1917" height="870" alt="image" src="https://github.com/user-attachments/assets/cca74a71-04ba-4a86-9785-bdb584db0cd0" />

