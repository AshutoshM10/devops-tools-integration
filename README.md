# Jenkins Pipeline Assignment

This document details the steps and tools used in the Jenkins pipeline setup to ensure code quality, security, and performance.

---

## **Objective**

To set up a Jenkins pipeline that integrates the following tasks:

- **Source Code Management**
- **Code Quality Analysis (SonarQube)**
- **Code Coverage (JaCoCo)**
- **Cyclomatic Complexity Analysis (Lizard)**
- **Security Vulnerability Scanning (OWASP Dependency-Check)**
- **Notification Handling**

---

## **Pipeline Stages Overview**

### 1. **Checkout Code**
- Pulls the source code from the Git repository configured in Jenkins.

### 2. **Code Quality Analysis**
- Leverages SonarQube to analyze code for bugs, vulnerabilities, and code smells.

### 3. **Code Coverage**
- Runs JaCoCo to ensure sufficient test coverage of the codebase.
- Publishes the JaCoCo HTML report to Jenkins.

### 4. **Cyclomatic Complexity Analysis**
- Utilizes Lizard to calculate cyclomatic complexity and stores the report for analysis.

### 5. **Security Vulnerability Scanning**
- Uses OWASP Dependency-Check to identify dependencies with known vulnerabilities.
- Publishes the OWASP report in Jenkins for review.

### 6. **Post Quality Gate**
- Verifies the SonarQube Quality Gate, ensuring the pipeline proceeds only if the quality gate is passed.

---

## **Tools Used**

1. **SonarQube**
   - Performs static code analysis.
   - Requires configuration in Jenkins with a server instance.

2. **JaCoCo**
   - Provides code coverage reports for Java applications.
   - Integrated via Maven.

3. **Lizard**
   - Calculates cyclomatic complexity for multiple programming languages.

4. **OWASP Dependency-Check**
   - Identifies vulnerabilities in project dependencies.

5. **Jenkins**
   - Orchestrates the entire pipeline.

---

## **Jenkinsfile**

```groovy
pipeline {
    agent any

    environment {
        SONARQUBE_SERVER = 'sonarqube url or ip:9000'
        SONARQUBE_PROJECT = 'our-project-key'
        OWASP_REPORT_DIR = 'owasp-dependency-check-report'
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Code Quality Analysis') {
            steps {
                withSonarQubeEnv('SonarQubeServer') {
                    sh """
                        mvn sonar:sonar \
                            -Dsonar.projectKey=${SONARQUBE_PROJECT} \
                            -Dsonar.host.url=$SONAR_HOST_URL \
                            -Dsonar.login=$SONAR_AUTH_TOKEN
                    """
                }
            }
        }

        stage('Code Coverage') {
            steps {
                sh 'mvn test'
                sh 'mvn jacoco:report'
                publishHTML(target: [
                    reportDir: 'target/site/jacoco',
                    reportFiles: 'index.html',
                    reportName: 'JaCoCo Coverage Report'
                ])
            }
        }

        stage('Cyclomatic Complexity') {
            steps {
                sh """
                    pip install lizard
                    lizard . > lizard-report.txt
                """
                archiveArtifacts artifacts: 'lizard-report.txt'
            }
        }

        stage('Security Scan') {
            steps {
                sh """
                    mkdir -p ${OWASP_REPORT_DIR}
                    dependency-check --scan . --out ${OWASP_REPORT_DIR}
                """
                publishHTML(target: [
                    reportDir: "${OWASP_REPORT_DIR}",
                    reportFiles: 'dependency-check-report.html',
                    reportName: 'OWASP Dependency Check Report'
                ])
            }
        }

        stage('Post Quality Gate') {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }

    post {
        success {
            emailext(
                to: 'team@example.com',
                subject: 'Jenkins Build Successful: ${env.JOB_NAME} #${env.BUILD_NUMBER}',
                body: """
                Build Successful:
                - Job: ${env.JOB_NAME}
                - Build Number: ${env.BUILD_NUMBER}
                - Build URL: ${env.BUILD_URL}
                """
            )
        }
        failure {
            emailext(
                to: 'team@example.com',
                subject: 'Jenkins Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}',
                body: """
                Build Failed:
                - Job: ${env.JOB_NAME}
                - Build Number: ${env.BUILD_NUMBER}
                - Build URL: ${env.BUILD_URL}
                """
            )
        }
    }
}
```

---

## **Notes**

1. **Prerequisites**:
   - Ensure Maven is installed and configured.
   - Set up SonarQube in Jenkins with appropriate credentials.
   - Install Python for Lizard.
   - Configure OWASP Dependency-Check in Jenkins or locally.

2. **Jenkins Plugins**:
   - SonarQube Scanner.
   - HTML Publisher.

3. **Environment Variables**:
   - `SONAR_HOST_URL` and `SONAR_AUTH_TOKEN` must be configured in Jenkins.

4. **Reports**:
   - JaCoCo: `target/site/jacoco/index.html`.
   - Lizard: `lizard-report.txt`.
   - OWASP Dependency-Check: `dependency-check-report.html`.

5. **Notifications**:
   - Set up `emailext` plugin in Jenkins for email notifications.

---

## **Next Steps**

- Ensure the pipeline runs end-to-end successfully.
- Review all generated reports for actionable insights.
- Share pipeline outputs with the team for feedback and improvements.

---

Feel free to update this README with additional findings or enhancements as you proceed!
