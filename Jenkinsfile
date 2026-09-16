@Library('jenkins-shared-library') _

def config

pipeline {
    agent any

    stages {

        stage('Read Config') {
            steps {
                script {
                    config = readProperties file: 'config.properties'
                }
            }
        }

        stage('Clone') {
            steps {
                cloneRepo(config.GIT_URL)
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