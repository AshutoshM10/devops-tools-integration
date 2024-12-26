pipeline {
    agent any

    environment {
        SONARQUBE_SERVER = 'sonar url or ip address'  
        SONARQUBE_PROJECT = 'our-project-key'
        OWASP_REPORT_DIR = 'owasp-dependency-check-report'
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo "Checking out code from Git repository"
                checkout scm
            }
        }

        stage('Code Quality Analysis') {
            steps {
                echo "Running SonarQube analysis"
                script {
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
        }

        stage('Code Coverage') {
            steps {
                echo "Running JaCoCo for code coverage"
                sh 'mvn test'
                sh 'mvn jacoco:report'
                publishHTML(target: [
                    allowMissing: false,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'target/site/jacoco',
                    reportFiles: 'index.html',
                    reportName: 'JaCoCo Coverage Report'
                ])
            }
        }

        stage('Cyclomatic Complexity') {
            steps {
                echo "Running Lizard for cyclomatic complexity analysis"
                sh """
                    pip install lizard
                    lizard . > lizard-report.txt
                """
                archiveArtifacts artifacts: 'lizard-report.txt', fingerprint: true
                echo "Cyclomatic complexity analysis complete. Report archived."
            }
        }

        stage('Security Scan') {
            steps {
                echo "Running OWASP Dependency-Check"
                sh """
                    mkdir -p ${OWASP_REPORT_DIR}
                    dependency-check --scan . --out ${OWASP_REPORT_DIR}
                """
                publishHTML(target: [
                    allowMissing: false,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: "${OWASP_REPORT_DIR}",
                    reportFiles: 'dependency-check-report.html',
                    reportName: 'OWASP Dependency Check Report'
                ])
            }
        }

        stage('Post Quality Gate') {
            steps {
                echo "Checking SonarQube Quality Gate"
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully"
            emailext(
                to: 'team@example.com',
                subject: 'Jenkins Build Successful: ${env.JOB_NAME} #${env.BUILD_NUMBER}',
                body: """
                Jenkins Build Successful:
                - Job: ${env.JOB_NAME}
                - Build Number: ${env.BUILD_NUMBER}
                - Build URL: ${env.BUILD_URL}
                """
            )
        }
        failure {
            echo "Pipeline failed"
            emailext(
                to: 'team@example.com',
                subject: 'Jenkins Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}',
                body: """
                Jenkins Build Failed:
                - Job: ${env.JOB_NAME}
                - Build Number: ${env.BUILD_NUMBER}
                - Build URL: ${env.BUILD_URL}
                """
            )
        }
    }
}
