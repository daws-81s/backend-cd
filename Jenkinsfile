pipeline {
    agent {
        label 'AGENT-1'
    }
    options{
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
        //retry(1)
    }
    parameters{
        choice(name: 'ENVIRONMENT', choices: ['dev', 'qa', 'uat', 'pre-prod', 'prod'], description: 'Select your Environment')
        string(name: 'version',  description: 'Enter your application version')
        string(name: 'jira-id',  description: 'Enter your jira id')
    }
    environment {
        appVersion = '' // this will become global, we can use across pipeline
        region = 'us-east-1'
        account_id = ''
        project = 'expense'
        environment = ''
        component = 'backend'
    }

    stages {
        
        stage('Setup Environment'){
            steps{
                script{
                    environment = params.ENVIRONMENT
                    appVersion = params.version
                    account_id = pipelineGlobals.getAccountID(environment)
                }
            }
        }
        stage('Integration tests'){
            when {
                expression {params.ENVIRONMENT == 'qa'}
            }
            steps{
                script{
                    sh """
                        echo "Run integration tests"
                    """
                }
            }
        }
        stage('Check JIRA'){
            when {
                expression {params.ENVIRONMENT == 'prod'}
            }
            steps{
                script{
                    sh """
                        echo "check jira status"
                        echo "check jira deployment window"
                        echo "fail pipeline if above two are not true"
                    """
                }
            }
        }
        stage('Deploy'){
            steps{
                withAWS(region: 'us-east-1', credentials: 'aws-creds') {
                    sh """
                        aws eks update-kubeconfig --region ${region} --name ${project}-dev
                        cd helm
                        sed -i 's/IMAGE_VERSION/${appVersion}/g' values-${environment}.yaml
                        helm upgrade --install ${component} -n ${project} -f values-${environment}.yaml .
                    """
                }
            }
        }
    }

    post {
        always{
            echo "This sections runs always"
            deleteDir()
        }
        success{
            echo "This section run when pipeline success"
        }
        failure{
            echo "This section run when pipeline failure"
        }
    }
}