pipeline {
    agent any

    environment {
        DOCKER_IMAGE     = "v-day-app:latest"
        DOCKERHUB_REPO   = "nro576809/v-day-app"
        AWS_REGION       = "us-east-1"
    }

    stages {

        stage('Clone Repo') {
            steps {
                git url: 'https://github.com/Websitesbuild/V-Day-2.1.git', branch: 'main'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'Docker',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker tag ${DOCKER_IMAGE} ${DOCKERHUB_REPO}:latest
                        docker push ${DOCKERHUB_REPO}:latest
                    """
                }
            }
        }

        stage('Terraform — Provision EC2') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'AWS'
                ]]) {
                    dir('terraform') {
                        sh """
                            terraform init
                            terraform apply -auto-approve
                        """
                    }
                }
            }
        }

        stage('Wait for EC2 to be Ready') {
            steps {
                sh "sleep 40"   // give instances time to boot
            }
        }

        stage('Ansible — Install Docker & Deploy') {
            steps {
                sh """
                    ansible-playbook -i ansible/inventory/hosts.ini ansible/site.yml
                """
            }
        }

    }

    post {
        success {
            echo '✅ Deployment successful on both EC2 instances!'
        }
        failure {
            echo '❌ Pipeline failed. Check logs above.'
        }
    }
}
