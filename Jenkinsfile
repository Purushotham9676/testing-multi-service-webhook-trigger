pipeline {
    agent any

    environment {
        PR_NUMBER = "${env.CHANGE_ID}"
        TARGET_BRANCH = "main"
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo "Checking out code for PR #${PR_NUMBER}."
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: 'main']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/Purushotham9676/testing-multi-service-webhook-trigger.git'
                    ]]
                ])
            }
        }

        stage('Identify Changed Services') {
            steps {
                script {
                    def changedDirs = sh(
                        script: "git diff --name-only origin/${TARGET_BRANCH}...HEAD | grep '/' | cut -d'/' -f1 | sort -u",
                        returnStdout: true
                    ).trim()

                    if (changedDirs) {
                        echo "Changes detected in the following services: ${changedDirs.split('\n').join(', ')}"
                        env.CHANGED_SERVICES = changedDirs
                    } else {
                        echo "No service changes detected. Skipping build stage."
                        env.CHANGED_SERVICES = ""
                    }
                }
            }
        }

        stage('Build Service Dockerfiles') {
            when {
                expression { env.CHANGED_SERVICES != "" }
            }
            steps {
                script {
                    def services = env.CHANGED_SERVICES.split('\n')
                    for (service in services) {
                        echo "--- Starting build for service: ${service} ---"

                        dir(service) {
                            if (fileExists('Dockerfile')) {
                                echo "Dockerfile found. Building image for ${service}..."
                                sh "docker build -t ${service}:${PR_NUMBER} ."
                                echo "Successfully built Docker image: ${service}:${PR_NUMBER}"
                            } else {
                                echo "WARNING: No Dockerfile found in '${service}'. Skipping build for this service."
                            }
                        }
                    }
                }
            }
        }
    }
}
