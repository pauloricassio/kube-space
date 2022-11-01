pipeline {
    agent any

    stages {

        stage ('Build Docker Image') {
            steps {
                script {
                    dockerapp = docker.build("p4ul0/kube-news:${env.BUILD_ID}", '-f ./src/Dockerfile ./src')
                }
            }
        }

        stage ('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                        dockerapp.push('latest')
                        dockerapp.push("${env.BUILD_ID}")

                    }
                }
            }
        }

        stage ('Deploy Kubernetes') {
            environment{
                tag_version = "${env.BUILD_ID}"
            }
            steps {
                withKubeConfig ([credentialsId: 'kubeconfigg']) {
                    sh 'sed -i "s/{{TAG}}/$tag_version/g" ./k8s/deployment.yml'
                    sh 'kubectl apply -f ./k8s/deployment.yml'
                }                
                
            }
        }

        stage ('Deploy Prometheus+Grafana') {
            steps {
                withKubeConfig ([credentialsId: 'kubeconfig']) {
                    sh 'kubectl apply -f ./monitoramento/deploy-prometheus-grafana.yml'
                }
            }
        }

        stage('Notificando sucesso') {
            steps {
                slackSend (color: 'good', message: '[ Sucesso ] O novo build esta disponivel em: http://146.190.198.194/ ', tokenCredentialId: 'slack-token')
            }
        } 

        stage('Notificando falha') {
            steps {
                sh 'sed -i "s/{{TAG}}/$tag_version/g" http://134.209.67.145:8080/job/kube-space/{{TAG}}/console/'
                slackSend failOnError:true (color: 'danger', message: '[ Erro ] Houve uma falha na execucao esteira, verifique: http://134.209.67.145:8080/job/kube-space/{{TAG}}/console/ ', tokenCredentialId: 'slack-token')
            }
        }     

    }
}