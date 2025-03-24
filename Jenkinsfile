pipeline {
    agent any

    environment {
        GIT_DEPLOYMENT_YAML = 'https://raw.githubusercontent.com/Shin-JiHyun/ATP_jenkins/develop/k8s/frontend-deployment.yaml'
        GIT_SERVICE_YAML = 'https://raw.githubusercontent.com/Shin-JiHyun/ATP_jenkins/develop/k8s/frontend-service.yaml'
        NAMESPACE = 'sjh'
        SSH_USER = 'test'  // SSH 서버 사용자
        SSH_HOST = '192.0.1.30'  // SSH 서버 IP
        SSH_KEY = '/var/lib/jenkins/.ssh/id_rsa'  // SSH Private Key 경로
    }

    stages {
        stage('SSH') {
			steps{
				script{
					sshPublisher(
                        publishers: [
                            sshPublisherDesc(
                                configName: 'k8s',
                                verbose: true,
                                transfers: [
                                    sshTransfer(
                                        execCommand: """
                            			curl -sL ${GIT_DEPLOYMENT_YAML} | 
                            			sed 's/latest/${BUILD_ID}/g' | 
                            			sed 's/canary/true/g' | 
                           			kubectl apply -n ${NAMESPACE} -f -
                                        """
                                    ),
                                    sshTransfer(
                                        execCommand: """
                           			kubectl rollout status deployment/frontend -n ${NAMESPACE}
                            			kubectl wait --for=condition=available deployment/front --timeout=120s -n ${NAMESPACE}
                                        """
                                    ),
                                    sshTransfer(
                                        execCommand: """
                            curl -sL ${GIT_SERVICE_YAML} | 
                            sed 's/canary/true/g' | 
                            kubectl apply -n ${NAMESPACE} -f -
                                        """
                                    ),

                                ]
                            )
                        ]
                    )
                }
            }
        }
    }
}