pipeline {
    agent any
    environment {
        DOCKER_IMAGE_NAME = "mayurgowda0511/train-schedule"
        CANARY_REPLICAS = 1
    }
    stages {
        // Build stage is commented out for simplicity

        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    // Define the Docker image name with a tag
                    def dockerImage = docker.image(DOCKER_IMAGE_NAME)
                    
                    // Build the Docker image
                    dockerImage.build()

                    // Tag the image with the build number and 'latest'
                    dockerImage.tag("${env.BUILD_NUMBER}")
                    dockerImage.tag("latest")
                }
            }
        }

        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    // Push the Docker image to the registry
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        docker.image(DOCKER_IMAGE_NAME).push()
                    }
                }
            }
        }

        stage('CanaryDeploy') {
            when {
                branch 'master'
            }
            steps {
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube-canary.yml',
                    enableConfigSubstitution: true
                )
            }
        }

        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                // Set CANARY_REPLICAS to 0 for production deployment
                script {
                    env.CANARY_REPLICAS = 0
                }
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube-canary.yml',
                    enableConfigSubstitution: true
                )
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube.yml',
                    enableConfigSubstitution: true
                )
            }
        }
    }
}
