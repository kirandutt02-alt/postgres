@Library('jenkins-shared-library') _

def config

pipeline {
    agent any

    parameters {
        choice(
            name: 'ENVIRONMENT',
            choices: ['dev', 'staging', 'prod'],
            description: 'Target environment to deploy'
        )
    }

    stages {

        stage('Read Config') {
            steps {
                script {
                    config = readProperties file: "config/${params.ENVIRONMENT}.properties"
                }
            }
        }

        stage('Clone') {
            steps {
                cloneRepo(config.GIT_URL)
            }
        }

        stage('Debug') {
            steps {
                sh 'pwd && find . -name "site.yml"'
            }
        }

        stage('User Approval') {
            when {
                expression {
                    config.KEEP_APPROVAL_STAGE.toBoolean()
                }
            }
            steps {
                userApproval(config.ENVIRONMENT)
            }
        }

        stage('Playbook Execution') {
            steps {
                executePlaybook(config.CODE_BASE_PATH)
            }
        }

        stage('Notification') {
            steps {
                sendNotification(
                    config.SLACK_CHANNEL_NAME,
                    config.ACTION_MESSAGE
                )
            }
        }
    }
}
