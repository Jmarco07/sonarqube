# SonarQube Local Setup Guide

A step-by-step guide to install, run, and scan your project locally using SonarQube.

---

## Prerequisites

| Tool | Version | Purpose |
|------|---------|---------|
| Java JDK | 17+ (recommended: JDK 21) | Required by SonarQube & Scanner |
| SonarQube | 10.7+ (Community Edition) | Code analysis server |
| SonarScanner CLI | 6.2+ | Scans your project |
| Docker (optional) | Latest | Alternative to manual install |

---

## Option A: Manual Installation

### Step 1 — Install Java JDK 21

```bash
# Install via Homebrew
brew install openjdk@21

# Add to PATH
echo 'export PATH="/opt/homebrew/opt/openjdk@21/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc

# Verify
java -version
```

Expected output: `openjdk version "21.x.x"`

---

### Step 2 — Download & Install SonarQube

```bash
# Download SonarQube Community Edition (latest)
cd ~/Downloads
curl -LO https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-10.7.0.96327.zip

# Extract and move
unzip sonarqube-10.7.0.96327.zip
mv sonarqube-10.7.0.96327 ~/sonarqube
```

---

### Step 3 — Start SonarQube Server

```bash
# macOS (Apple Silicon / Intel)
~/sonarqube/bin/macosx-universal-64/sonar.sh start

# Check status
~/sonarqube/bin/macosx-universal-64/sonar.sh status
```

Wait ~1-2 minutes, then open: **http://localhost:9000**

Default credentials:
- **Username:** `admin`
- **Password:** `admin`

> You'll be prompted to change the password on first login.

---

### Step 4 — Install SonarScanner CLI

```bash
brew install sonar-scanner

# Verify
sonar-scanner --version
```

---

### Step 5 — Create a Project in SonarQube

1. Open **http://localhost:9000**
2. Click **"Create Project"** → **"Manually"**
3. Enter:
   - Project key: `customer-api`
   - Display name: `Customer API`
4. Click **"Set Up"**
5. Select **"Locally"**
6. Generate a **token** (e.g., `my-local-token`) — copy it

---

### Step 6 — Run the Scan

From your project root:

```bash
cd ~/Documents/customerAPI

sonar-scanner \
  -Dsonar.projectKey=customer-api \
  -Dsonar.sources=. \
  -Dsonar.host.url=http://localhost:9000 \
  -Dsonar.token=<YOUR_GENERATED_TOKEN>
```

Replace `<YOUR_GENERATED_TOKEN>` with the token from Step 5.

---

### Step 7 — View Results

Open **http://localhost:9000/dashboard?id=customer-api** to see:
- Bugs
- Vulnerabilities
- Code Smells
- Coverage
- Duplications

---

## Option B: Docker Installation (Recommended)

### Step 1 — Run SonarQube with Docker

```bash
docker run -d \
  --name sonarqube \
  -p 9000:9000 \
  sonarqube:10.7-community
```

Wait ~1-2 minutes, then open **http://localhost:9000** (admin/admin).

---

### Step 2 — Install SonarScanner CLI

```bash
brew install sonar-scanner
```

---

### Step 3 — Create Project & Run Scan

Follow **Steps 5-7** from Option A above.

---

## Configuration File (Optional)

Create a `sonar-project.properties` file in your project root to avoid passing flags every time:

```properties
sonar.projectKey=customer-api
sonar.projectName=Customer API
sonar.projectVersion=1.0

sonar.sources=.
sonar.host.url=http://localhost:9000
sonar.token=<YOUR_GENERATED_TOKEN>

# Exclude non-source files
sonar.exclusions=**/node_modules/**,**/*.md,**/venv/**,**/__pycache__/**

# Python-specific (if applicable)
sonar.python.version=3.11

# Encoding
sonar.sourceEncoding=UTF-8
```

Then simply run:

```bash
sonar-scanner
```

---

## Stopping SonarQube

**Manual install:**
```bash
~/sonarqube/bin/macosx-universal-64/sonar.sh stop
```

**Docker:**
```bash
docker stop sonarqube
docker start sonarqube  # to restart later
```

---

## Troubleshooting

| Issue | Fix |
|-------|-----|
| `java.lang.IllegalStateException` on startup | Ensure JDK 17+ is installed and in PATH |
| Port 9000 already in use | Change port in `~/sonarqube/conf/sonar.properties`: `sonar.web.port=9001` |
| Scanner can't connect to server | Ensure SonarQube is fully started (check http://localhost:9000) |
| Elasticsearch error on macOS | Run: `sudo sysctl -w kern.maxproc=2048` |
| Permission denied on `sonar.sh` | Run: `chmod +x ~/sonarqube/bin/macosx-universal-64/sonar.sh` |

---

## Useful Commands

```bash
# Check SonarQube logs
tail -f ~/sonarqube/logs/sonar.log

# Check scanner debug output
sonar-scanner -X

# Docker logs
docker logs -f sonarqube
```
