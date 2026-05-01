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
                        """
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