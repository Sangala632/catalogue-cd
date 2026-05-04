pipeline {
    agent {
        label 'agent-1'
    } 
    environment {
        appVersion = ""
        REGION = "us-east-1"
        ACC_ID = "838180513114"
        PROJECT = "roboshop"
        COMPONENT = "catalogue"
    }
    options {
        timeout(time: 30, unit: 'MINUTES') 
        disableConcurrentBuilds()
    }
    parameters {
        string(name: 'appVersion', description: 'Image version of the application')
        choice(name: 'deploy_to', choices: ['dev', 'qa', 'prod'], description: 'Pick the Environment')
    }
    stages {
        stage('Deploy') {
            steps {
                script {
                    withAWS(credentials: 'aws-cerds', region: "${REGION}") {
                        sh """
                            aws eks update-kubeconfig --region $REGION --name "$PROJECT-${params.deploy_to}"
                            kubectl get nodes
                            kubectl apply -f NameSpace.yaml
                            sed -i "s/IMAGE_VERSION/${params.appVersion}/g" values-${params.deploy_to}.yaml
                            helm upgrade --install $COMPONENT -f values-${params.deploy_to}.yaml -n $PROJECT .
                        """
                    }
                }
            }
        }

        stage('Check Status'){
            steps{
                script {
    withAWS(credentials: 'aws-creds', region: 'us-east-1') {
        def deploymentStatus = sh(
            returnStdout: true,
            script: "kubectl rollout status deployment/$COMPONENT --timeout=30s -n $PROJECT || echo FAILED"
        ).trim()

        if (deploymentStatus.contains("successfully rolled out")) {
            echo "Deployment is success"
        } else {
            echo "Deployment failed! Triggering rollback..."
            sh "helm rollback $COMPONENT -n $PROJECT"
            sleep(20)

            def rollbackStatus = sh(
                returnStdout: true,
                script: "kubectl rollout status deployment/$COMPONENT --timeout=30s -n $PROJECT || echo FAILED"
            ).trim()

            if (rollbackStatus.contains("successfully rolled out")) {
                error "Deployment is Failure, Rollback Success"
            } else {
                error "Deployment is Failure, Rollback Failure. Application is not running"
            }
        }
    }
}
            }
        }

    }

    post { 
        always { 
            echo 'I will always say Hello again!'
        }

        success { 
            echo 'If success say ..I will always say Hello again!'
        }

        
        failure { 
            echo 'If failure say ..I will always say stage is failure!'
        }
    }

}