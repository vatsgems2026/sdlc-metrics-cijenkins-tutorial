# SDLC Metrics Jenkins Demo

**Companion repository for the CloudBees Professional Services adoption journey guide: "Populating SDLC Metrics with Jenkins and CI"**

This repository provides a complete, working example of how to instrument a Jenkins pipeline to populate all SDLC metrics dashboards in CloudBees Unify, including:

- **CI Insights** - Jenkins operational health metrics
- **Software Delivery Activity** - Build and deployment tracking
- **DORA Metrics** - Deployment frequency, lead time, change failure rate, MTTR
- **Flow Metrics** - Work item tracking with Jira integration
- **Security Insights** - SAST and SCA vulnerability scanning
- **Test Insights** - Unit test result aggregation
- **Environment Inventory** - Artifact version tracking

## Two-Branch Learning Approach

This repository uses **two branches** to support different learning paths:

### Branch: `clean` (START HERE)
Basic CI/CD pipeline **WITHOUT CloudBees instrumentation**. Use this branch to:
- Follow the adoption guide step-by-step
- Add CloudBees integration incrementally
- Watch each dashboard populate as you add instrumentation
- Learn what each integration point does

### Branch: `complete` (REFERENCE)
Fully instrumented pipeline with **ALL CloudBees integration**. Use this branch to:
- Compare your work with the complete implementation
- Troubleshoot when something doesn't work
- See the final target state

**For consultants following the adoption guide:** Start with the `clean` branch and see **[SETUP.md](SETUP.md)** for detailed instructions.

**For viewing a complete working example:** Use the `complete` branch and continue reading this README.

## Project Structure

```
sdlc-metrics-jenkins-demo/
├── src/
│   ├── c/                      # C application source code
│   │   ├── calculator.c        # Calculator implementation
│   │   ├── calculator.h        # Calculator header
│   │   └── main.c              # Main application entry point
│   └── python/                 # Python Flask API
│       └── app.py              # Flask web application
├── tests/
│   ├── c/                      # C unit tests
│   │   └── test_calculator.c   # Calculator test suite
│   └── python/                 # Python unit tests
│       └── test_app.py         # Flask API test suite
├── Dockerfile                  # Multi-stage Docker build
├── Makefile                    # C application build configuration
├── Jenkinsfile                 # Complete CI/CD pipeline with SDLC metrics
├── requirements.txt            # Python dependencies
├── sonar-project.properties    # SonarQube configuration
├── pytest.ini                  # Pytest configuration
└── README.md                   # This file
```

## What This Demo Includes

### Applications

**C Calculator Application:**
- Command-line calculator with basic arithmetic operations
- Demonstrates C compilation and testing in Jenkins
- Generates JUnit XML test results

**Python Flask API:**
- RESTful calculator API
- Health check endpoint for monitoring
- Demonstrates Python application deployment
- Comprehensive pytest test coverage

### Jenkins Pipeline Features

The `Jenkinsfile` demonstrates **all CloudBees Unify SDLC metrics integration points**:

1. **Build Tracking**
   - C application compilation with `make`
   - Python application validation
   - Docker image building and tagging

2. **Test Result Publishing**
   - C unit tests with JUnit XML output
   - Python pytest with JUnit XML reporting
   - Automatic ingestion via `junit` step

3. **Security Scanning**
   - SAST with SonarQube integration (`exportSonarQubeScan`)
   - SCA with Trivy filesystem scanning
   - Container scanning with Trivy image analysis
   - SARIF format reporting (`registerSecurityScan`)

4. **Artifact Registration**
   - Docker image metadata with `registerBuildArtifactMetadata`
   - Build artifact tracking with digest/hash
   - Version labeling and tagging

5. **Deployment Tracking**
   - Environment-based deployment (Development/Production)
   - DORA metrics population
   - Deployment success/failure tracking

## Prerequisites

### Required Tools

- **Jenkins** 2.414.3+ (open-source) or CloudBees CI 2.440.3.7+
- **Docker** 20.10+
- **Git**
- **GCC** (for C compilation)
- **Python 3.11+**
- **Make**

### Jenkins Plugins Required

- CloudBees Platform Insights Plugin
- JUnit Plugin
- Docker Pipeline Plugin
- Git Plugin
- Pipeline Plugin
- Credentials Plugin

### CloudBees Unify Setup

Before using this demo, complete the following in CloudBees Unify:

1. Install and authenticate CloudBees Platform Insights plugin in Jenkins
2. Create environments (Development, Staging, Production)
3. Configure Jira integration for Flow Metrics (optional)
4. Set up Docker registry credentials
5. Configure SonarQube integration (optional)

**See the adoption journey guide for detailed setup instructions.**

## Quick Start

**Are you following the adoption guide?**
- **YES**: See **[SETUP.md](SETUP.md)** to set up the `clean` branch and follow along
- **NO**: Continue below for complete implementation quick start

### 1. Clone the Repository

```bash
git clone https://github.com/YOUR_ORG/sdlc-metrics-jenkins-demo.git
cd sdlc-metrics-jenkins-demo

# For following the guide: use clean branch
git checkout clean

# For complete reference: use complete branch
git checkout complete
```

### 2. Local Build and Test

Test the applications locally before running in Jenkins:

**C Application:**
```bash
make clean
make all
make test
./build/calculator
```

**Python Application:**
```bash
pip install -r requirements.txt
python3 src/python/app.py

# In another terminal:
curl http://localhost:5000/health
```

**Run Python Tests:**
```bash
pytest tests/python/ --verbose
```

**Build Docker Image:**
```bash
docker build -t sdlc-demo-app:local .
docker run -p 5000:5000 sdlc-demo-app:local
```

### 3. Configure Jenkins Credentials

Before running the pipeline, configure these Jenkins credentials:

| Credential ID | Type | Description |
|---------------|------|-------------|
| `docker-registry-url` | Secret text | Docker registry URL (e.g., `docker.io/myorg`) |
| `docker-registry-credentials` | Username/Password | Docker Hub credentials |
| `sonarqube-url` | Secret text | SonarQube server URL |
| `sonarqube-token` | Secret text | SonarQube API token |

**To add credentials in Jenkins:**
1. Navigate to **Manage Jenkins > Manage Credentials**
2. Click **(global)** domain
3. Click **Add Credentials**
4. Enter the credential details
5. Click **Create**

### 4. Create Jenkins Pipeline Job

1. In Jenkins, click **New Item**
2. Enter name: `sdlc-metrics-demo`
3. Select **Multibranch Pipeline**
4. Click **OK**
5. Under **Branch Sources**, add your Git repository
6. Click **Save**
7. Jenkins will scan for branches and trigger builds

### 5. Verify SDLC Metrics in CloudBees Unify

After the pipeline runs successfully, verify data appears in CloudBees Unify dashboards:

- **CI Insights** (`Jenkins management > CI insights for Jenkins`)
- **Software Delivery Activity** (`Analytics > Software delivery activity`)
- **DORA Metrics** (`Analytics > DORA metrics`)
- **Flow Metrics** (`Analytics > DORA and flow metrics insights`)
- **Security Insights** (`Analytics > Security insights`)
- **Test Insights** (`Analytics > Test insights`)
- **Environment Inventory** (Component page > `Environment inventory` tab)

## Jenkinsfile Walkthrough

The `Jenkinsfile` is organized into the following stages:

### Build Stages
- **Initialize** - Set environment variables and Git info
- **Checkout** - Clone repository
- **Setup Environment** - Create directories and verify tools
- **Install Dependencies** - Install Python packages
- **Build C Application** - Compile C code with Make
- **Build Python Application** - Validate Python syntax

### Test Stages
- **Run C Unit Tests** - Execute C tests, generate JUnit XML
- **Run Python Unit Tests** - Execute pytest, generate JUnit XML
- **Code Quality Analysis** - Run flake8 linting

### Security Stages
- **SAST Security Scan** - SonarQube static analysis
- **SCA Security Scan** - Trivy filesystem vulnerability scan
- **Scan Docker Image** - Trivy container image scan

### Deployment Stages
- **Build Docker Image** - Multi-stage Docker build
- **Push Docker Image** - Push to registry (main/develop only)
- **Deploy to Environment** - Deploy based on branch
- **Smoke Tests** - Post-deployment validation

### CloudBees Integration Points

Each stage includes CloudBees-specific steps:

```groovy
// Test results
junit '**/test-results/*.xml'

// Security scans
registerSecurityScan(
    artifacts: "trivy-report.sarif",
    format: "sarif",
    scanner: "Trivy",
    archive: true
)

// Build artifacts
registerBuildArtifactMetadata(
    name: "${APP_NAME}",
    url: "${DOCKER_IMAGE}",
    version: "${APP_VERSION}",
    type: "Docker",
    digest: "${DOCKER_DIGEST}",
    label: "build-${BUILD_NUMBER}"
)

// SonarQube integration
exportSonarQubeScan(
    component: "",
    project: "${APP_NAME}",
    host: "${SONAR_HOST}",
    credentialId: "sonarqube-token"
)
```

## Customization

### Modify for Your Organization

**Update Docker Registry:**
```groovy
// In Jenkinsfile
DOCKER_REGISTRY = credentials('docker-registry-url')
```

**Update SonarQube Configuration:**
```properties
# In sonar-project.properties
sonar.projectKey=your-project-key
sonar.projectName=Your Project Name
```

**Change Application Name:**
```groovy
// In Jenkinsfile
APP_NAME = "your-app-name"
```

**Adjust Environment Mapping:**
```groovy
// In Jenkinsfile
DEPLOY_ENV = "${env.BRANCH_NAME == 'production' ? 'Production' : 'Staging'}"
```

### Add Additional Tests

**C Tests:**
Add test functions to `tests/c/test_calculator.c`:
```c
TEST(test_your_new_function) {
    ASSERT_EQUAL(expected, your_function(input));
}

// In main():
RUN_TEST(test_your_new_function);
```

**Python Tests:**
Add test methods to `tests/python/test_app.py`:
```python
def test_your_new_endpoint(client):
    response = client.get('/your/endpoint')
    assert response.status_code == 200
```

### Add More Security Scanners

The Jenkinsfile can be extended with additional scanners:

**Bandit (Python SAST):**
```groovy
stage('Python SAST - Bandit') {
    steps {
        sh 'bandit -r src/python/ -f json -o bandit-report.json'
    }
    post {
        always {
            registerSecurityScan(
                artifacts: "bandit-report.json",
                format: "json",
                scanner: "Bandit",
                archive: true
            )
        }
    }
}
```

**Snyk (SCA):**
```groovy
stage('SCA - Snyk') {
    steps {
        sh 'snyk test --json-file-output=snyk-report.json'
    }
    post {
        always {
            registerSecurityScan(
                artifacts: "snyk-report.json",
                format: "json",
                scanner: "Snyk",
                archive: true
            )
        }
    }
}
```

## Troubleshooting

### Build Failures

**C Compilation Errors:**
```bash
# Test locally first
make clean
make all
# Check for syntax errors in src/c/*.c
```

**Python Test Failures:**
```bash
# Run tests locally to see detailed error messages
pytest tests/python/ --verbose --tb=short
```

### Docker Issues

**Docker Build Fails:**
```bash
# Test Docker build locally
docker build -t test-image .

# Check Dockerfile syntax
docker build --no-cache -t test-image .
```

**Registry Push Fails:**
- Verify credentials are configured correctly in Jenkins
- Ensure Jenkins agent has Docker access
- Check network connectivity to registry

### Security Scanning Issues

**Trivy Installation Fails:**
- Check internet connectivity from Jenkins agent
- Verify wget/curl are available
- Manually install Trivy on the agent

**SonarQube Connection Fails:**
- Verify SonarQube URL is accessible from Jenkins
- Check SonarQube token is valid
- Ensure sonar-scanner is installed on Jenkins agent

### CloudBees Metrics Not Appearing

**Data Not in Dashboards:**
1. Verify CloudBees Platform Insights plugin is installed and authenticated
2. Check Jenkins logs for errors: `Manage Jenkins > System Log`
3. Wait 10-15 minutes for data ingestion
4. Verify pipeline steps completed successfully (check Console Output)

**Security Scans Not Showing:**
- Verify SARIF/JSON files are generated correctly
- Check `registerSecurityScan` step ran without errors
- Ensure scanner name matches CloudBees supported scanners

**Test Results Missing:**
- Verify JUnit XML files exist in `test-results/` directory
- Check `junit` step output for errors
- Ensure XML format is valid (use `xmllint` to validate)

## Support and Resources

- **Adoption Journey Guide**: See the CloudBees PS documentation for the complete step-by-step guide
- **CloudBees Documentation**: https://docs.cloudbees.com/docs/cloudbees-unify/latest/
- **Jenkins Documentation**: https://www.jenkins.io/doc/
- **Trivy Documentation**: https://aquasecurity.github.io/trivy/
- **SonarQube Documentation**: https://docs.sonarqube.org/

## Contributing

This is a demo repository for CloudBees Professional Services. For issues or improvements:

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

## License

This demo application is provided as-is for educational purposes.

## Version History

- **1.0.0** - Initial release with complete SDLC metrics integration

---

**Note:** This repository is designed to work with the CloudBees Professional Services adoption journey guide: "Populating SDLC Metrics with Jenkins and CI". Follow that guide for detailed explanations of each metric type and CloudBees Unify configuration.
