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
                    String configFile = "config/${params.ENVIRONMENT}.properties"

                    if (!fileExists(configFile)) {
                        error "Config file not found: ${configFile}"
                    }

                    config = readProperties file: configFile

                    if (config.ENVIRONMENT != params.ENVIRONMENT) {
                        error "Mismatch: params.ENVIRONMENT='${params.ENVIRONMENT}' " +
                              "but ${configFile} declares ENVIRONMENT='${config.ENVIRONMENT}'"
                    }
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

    post {
        failure {
            script {
                if (config) {
                    sendNotification(config.SLACK_CHANNEL_NAME, "${config.ACTION_MESSAGE} — FAILED")
                }
            }
        }
        aborted {
            script {
                if (config) {
                    sendNotification(config.SLACK_CHANNEL_NAME, "${config.ACTION_MESSAGE} — ABORTED (approval denied or timed out)")
                }
            }
        }
    }
}
