pipeline {
    agent any

    environment {
        GIT_DEPLOYMENT_YAML = 'https://raw.githubusercontent.com/Shin-JiHyun/ATP_jenkins/develop/k8s/frontend-deployment.yaml'
        GIT_CANARY_DEPLOYMENT_YAML = 'https://raw.githubusercontent.com/Shin-JiHyun/ATP_jenkins/develop/k8s/frontend-canary-deployment.yaml'
        GIT_SERVICE_YAML = 'https://raw.githubusercontent.com/Shin-JiHyun/ATP_jenkins/develop/k8s/frontend-service.yaml'
        GIT_CANARY_SERVICE_YAML = 'https://raw.githubusercontent.com/Shin-JiHyun/ATP_jenkins/develop/k8s/frontend-canary-service.yaml'
        GIT_INGRESS_YAML = 'https://raw.githubusercontent.com/Shin-JiHyun/ATP_jenkins/develop/k8s/frontend-ingress.yaml'
        GIT_CANARY_INGRESS_YAML = 'https://raw.githubusercontent.com/Shin-JiHyun/ATP_jenkins/develop/k8s/frontend-canary-ingress.yaml'

        NAMESPACE = 'sjh'
        SSH_USER = 'test'  // SSH 서버 사용자
        SSH_HOST = '192.0.1.30'  // SSH 서버 IP
        SSH_KEY = '/var/lib/jenkins/.ssh/id_rsa'  // SSH Private Key 경로
    }

    stages {
        stage('Deploy Services & Ingress') {
            steps {
                script {
                    sshPublisher(
                        publishers: [
                            sshPublisherDesc(
                                configName: 'k8s',
                                verbose: true,
                                transfers: [
                                    // 기존 배포
                                    sshTransfer(
                                        execCommand: """
                                            curl -sL ${GIT_DEPLOYMENT_YAML} | 
                                            sed 's/latest/${BUILD_ID}/g' | 
                                            sed 's/canary/false/g' | 
                                            kubectl apply -n ${NAMESPACE} -f -
                                        """
                                    ),
                                    // Canary 배포
                                    sshTransfer(
                                        execCommand: """
                                            curl -sL ${GIT_CANARY_DEPLOYMENT_YAML} | 
                                            sed 's/latest/${BUILD_ID}/g' | 
                                            sed 's/canary/true/g' | 
                                            kubectl apply -n ${NAMESPACE} -f -
                                        """
                                    ),
                                    // 기존 서비스 배포
                                    sshTransfer(
                                        execCommand: """
                                            curl -sL ${GIT_SERVICE_YAML} | 
                                            sed 's/canary/false/g' | 
                                            kubectl apply -n ${NAMESPACE} -f -
                                        """
                                    ),
                                    // Canary 서비스 배포
                                    sshTransfer(
                                        execCommand: """
                                            curl -sL ${GIT_CANARY_SERVICE_YAML} | 
                                            sed 's/canary/true/g' | 
                                            kubectl apply -n ${NAMESPACE} -f -
                                        """
                                    ),
                                    // 기존 Ingress 배포
                                    sshTransfer(
                                        execCommand: """
                                            curl -sL ${GIT_INGRESS_YAML} | 
                                            kubectl apply -n ${NAMESPACE} -f -
                                        """
                                    ),
                                    // Canary Ingress 배포
                                    sshTransfer(
                                        execCommand: """
                                            curl -sL ${GIT_CANARY_INGRESS_YAML} | 
                                            kubectl apply -n ${NAMESPACE} -f -
                                        """
                                    )
                                ]
                            )
                        ]
                    )
                }
            }
        }
        
        stage('Check Deployment Status') {
            steps {
                script {
                    sshPublisher(
                        publishers: [
                            sshPublisherDesc(
                                configName: 'k8s',
                                verbose: true,
                                transfers: [
                                    // 기존 배포 상태 확인
                                    sshTransfer(
                                        execCommand: """
                                            kubectl rollout status deployment/frontend -n ${NAMESPACE}
                                            kubectl wait --for=condition=available deployment/frontend --timeout=120s -n ${NAMESPACE}
                                        """
                                    ),
                                    // Canary 배포 상태 확인
                                    sshTransfer(
                                        execCommand: """
                                            kubectl rollout status deployment/frontend-canary -n ${NAMESPACE}
                                            kubectl wait --for=condition=available deployment/frontend-canary --timeout=120s -n ${NAMESPACE}
                                        """
                                    )
                                ]
                            )
                        ]
                    )
                }
            }
        }
    }
}
