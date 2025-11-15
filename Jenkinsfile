'''
pipeline {
    agent any

    environment {
        // This pipeline assumes you are using a Jenkins plugin like "GitHub Branch Source",
        // which automatically provides the pull request number in the CHANGE_ID variable.
        PR_NUMBER = "${env.CHANGE_ID}"
        
        // Define the target branch for comparison, e.g., \'main\' or \'master\'.
        TARGET_BRANCH = "main" 
    }

    stages {
        stage(\'Checkout Code\') {
            steps {
                // Step 1: Checks out the source code from the pull request branch.
                echo "Checking out code for PR #${PR_NUMBER}."
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: 'main']], // Change branch if needed
                    userRemoteConfigs: [[
                        url: 'https://github.com/Purushotham9676/testing-multi-service-webhook-trigger.git'
                    ]]
                ])
            }
        }

        stage(\'Identify Changed Services\') {
            steps {
                script {
                    // This command compares the PR branch with the target branch to find changed files.
                    // It then extracts the unique parent directory names (your service names).
                    def changedDirs = sh(
                        script: "git diff --name-only origin/${TARGET_BRANCH}...HEAD | grep \'/\' | cut -d\'/\' -f1 | sort -u",
                        returnStdout: true
                    ).trim()

                    if (changedDirs) {
                        echo "Changes detected in the following services: ${changedDirs.split(\'\\n\').join(\', \')}"
                        // Pass the list of changed services to the next stage.
                        env.CHANGED_SERVICES = changedDirs
                    } else {
                        echo "No service changes detected. Skipping build stage."
                        env.CHANGED_SERVICES = ""
                    }
                }
            }
        }

        stage(\'Build Service Dockerfiles\') {
            // This stage only runs if changes were detected in service directories.
            when {
                expression { env.CHANGED_SERVICES != "" }
            }
            steps {
                script {
                    def services = env.CHANGED_SERVICES.split(\'\\n\')
                    
                    // Loop through each identified service directory.
                    for (service in services) {
                        echo "--- Starting build for service: ${service} ---"
                        
                        // Change into the service\'s directory.
                        dir(service) {
                            // Verify that a Dockerfile exists in the directory.
                            if (fileExists(\'Dockerfile\')) {
                                echo "Dockerfile found. Building image for ${service}..."
                                
                                // Step 2: Build the Docker image and tag it with the PR number.
                                // The image name will be the service name, e.g., \'item-service:123\'.
                                sh "docker build -t ${service}:${PR_NUMBER} ."
                                echo "Successfully built Docker image: ${service}:${PR_NUMBER}"
                            } else {
                                echo "WARNING: No Dockerfile found in \'${service}\'. Skipping build for this service."
                            }
                        }
                    }
                }
            }
        }
    }

    // post {
    //     always {
    //         // This block runs after all stages to clean up.
    //         echo "Pipeline finished."
    //         // Optional: You can add steps here to clean up Docker images to save space on the Jenkins agent.
    //         script {
    //             if (env.CHANGED_SERVICES) {
    //                 def services = env.CHANGED_SERVICES.split(\'\\n\')
    //                 for (service in services) {
    //                     echo "Cleaning up image: ${service}:${PR_NUMBER}"
    //                     sh "docker rmi ${service}:${PR_NUMBER}"
    //                 }
    //             }
    //         }
    //     }
    // }
}
\'\'\'
