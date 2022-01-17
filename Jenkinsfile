pipeline {
    agent any
    environment {   
        DOCKER_IMAGE_NAME = "rishi001/train-schedule"
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
        stage('CanaryDeploy') {
            environment { 
                CANARY_REPLICAS = 0
            }
            steps {
                withEnv(['KUBECONFIG=${HOME}/.kube/admin.conf']) {
                    echo "KUBECONFIG is: $KUBECONFIG"
                }
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube-canary.yml',
                    enableConfigSubstitution: true
                )
            }
        }
        stage('DeployToProduction') {
            environment { 
                CANARY_REPLICAS = 0
            }
            steps {
               // input 'Deploy to Production?'
                //milestone(1)
                withEnv(['KUBECONFIG=${HOME}/.kube/admin.conf']) {
                    echo "KUBECONFIG is: $KUBECONFIG"
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
