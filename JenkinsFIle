pipeline {
    agent any
    environment {
        DISCORD_URL = credentials('discord')
        DOCKER_LOGIN = credentials('docker-gcp')
    }
    stages {
        stage('Checkout') {
            steps {
        // Checkout source code from GitHub
                git 'https://github.com/mfakhriabdillah/bp-dev.git'
            }
        }
        stage('Building Backend') {
            steps {
                echo 'Build Backend'
                sh 'docker build -t us-central1-docker.pkg.dev/studidevops-bigpro-fakhri/bp-studidevops/app-be:prod ./backend'
            }
        }
        stage('Building Frontend') {
            steps {
                echo 'Build Frontend'
                sh 'docker build -t us-central1-docker.pkg.dev/studidevops-bigpro-fakhri/bp-studidevops/app-fe:prod ./frontend'
            }
        }
        stage('Push Images'){
            steps{
                sh 'docker push us-central1-docker.pkg.dev/studidevops-bigpro-fakhri/bp-studidevops/app-be:prod'
                sh 'docker push us-central1-docker.pkg.dev/studidevops-bigpro-fakhri/bp-studidevops/app-fe:prod'
            }
        }
        stage('Update Backend Deployment YAML') {
            steps {
                dir('kubernetes/prod'){
                    script {
                        // Read the deployment YAML file
                        def deploymentYaml = readFile 'be-deployment-prod.yml'

                        // Update the image tag in the deployment YAML
                        deploymentYaml = deploymentYaml.replaceAll(/image:.*$/, "image: us-central1-docker.pkg.dev/studidevops-bigpro-fakhri/bp-studidevops/app-be:prod")

                        // Write the updated deployment YAML back to the file
                        writeFile file: 'be-deployment-prod.yml', text: deploymentYaml
                    }
                }
            }
        }
        stage('Update Frontend Deployment YAML') {
            steps {
                dir('kubernetes/prod'){
                    script {
                        // Read the deployment YAML file
                        def deploymentYaml = readFile 'fe-deployment-prod.yml'

                        // Update the image tag in the deployment YAML
                        deploymentYaml = deploymentYaml.replaceAll(/image:.*$/, "image: us-central1-docker.pkg.dev/studidevops-bigpro-fakhri/bp-studidevops/app-fe:prod")

                        // Write the updated deployment YAML back to the file
                        writeFile file: 'fe-deployment-prod.yml', text: deploymentYaml
                    }
                }
            }
        }
        stage('Printing conf') {
            steps {
                dir('kubernetes/prod') {
                    echo "kita lihat dulu"
                    sh 'cat be-deployment-prod.yml'
                    sh 'cat fe-deployment-prod.yml'
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                dir('kubernetes/prod') {
                    sh 'kubectl apply -f be-deployment-prod.yml -n production'
                    sh 'kubectl apply -f fe-deployment-prod.yml -n production'
                }
            }
            post { 
                aborted { 
                    discordSend description: "Jenkins Pipeline ${BUILD_NUMBER}", footer: "Build aborted", link: env.BUILD_URL, result: currentBuild.currentResult, title: JOB_NAME, webhookURL: $DISCORD_URL
                }
                success { 
                    discordSend description: "Jenkins Pipeline ${BUILD_NUMBER}", footer: "Build Success", link: env.BUILD_URL, result: currentBuild.currentResult, title: JOB_NAME, webhookURL: $DISCORD_URL
                }
                failure { 
                    discordSend description: "Jenkins Pipeline ${BUILD_NUMBER}", footer: "Build Failure", link: env.BUILD_URL, result: currentBuild.currentResult, title: JOB_NAME, webhookURL: $DISCORD_URL
                }
            }
    }
        }
        
}