pipeline {
    agent any

    environment {
        PR_NUMBER = "${env.CHANGE_ID ?: 'No PR Number'}"
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
                // Optional: Verify the current git status and fetch latest changes
                sh 'git status'
            }
        }

        stage('Identify Changed Services') {
            steps {
                script {
                    // Fetch the latest commit hash of the target branch (main)
                    def latestTargetCommit = sh(script: "git rev-parse origin/${TARGET_BRANCH}", returnStdout: true).trim()

                    // Compare the PR branch with the latest commit from the target branch (main)
                    def changedDirs = sh(
                        script: "git diff --name-only ${latestTargetCommit}...HEAD | grep '/' | cut -d'/' -f1 | sort -u",
                        returnStdout: true
                    ).trim()

                    // If there are any changes, assign them to the environment variable
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
                    // Split the changed services and iterate over them
                    def services = env.CHANGED_SERVICES.split('\n')
                    for (service in services) {
                        echo "--- Starting build for service: ${service} ---"

                        // Check if Dockerfile exists and build the image
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
