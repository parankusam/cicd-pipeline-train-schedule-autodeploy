pipeline {
    agent any
    environment {
        
        DOCKER_IMAGE_NAME = "parankusam/train-schedule"
    }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    app = docker.build(DOCKER_IMAGE_NAME)
                    app.inside {
                        sh 'echo Hello, World!'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('CanaryDeploy') {
            environment { 
                CANARY_REPLICAS = 1
            }
            steps {
               sh 'ansible-playbook -i /etc/ansible/hosts kube-ansible.yml --extra-vars "env=test build=$BUILD_NUMBER"'
            }
        }
        stage('DeployToProduction') {
            environment { 
                CANARY_REPLICAS = 0
            }
            steps {
               sh 'ansible-playbook -i /etc/ansible/hosts kube-ansible.yml --extra-vars "env=prod build=$BUILD_NUMBER"'
                 }
        }
    }
}
