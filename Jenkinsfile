pipeline {
    agent {
        node {
            label 'AGENT-1'
        }
    }
    environment {
        COURSE = "Jenkins"
        appVersion = ""
    }
    options {
        timeout(time: 10, unit: 'MINUTES') 
        disableConcurrentBuilds()
    }


    // This is build sections
    stages {
        stage('Read Version') {
            steps {
                script {
                    def packageJSON = readJSON file: 'package.json'
                    appVersion = packageJSON.version
                    echo "app version: ${appVersion}"
                }
            }
        }

        stage('install dependencies') {
            steps {
                script {
                    sh """
                        npm install
                    """
                }
            }
        }

         stage('build image') {
            steps {
                script {
                    sh """
                       sh "docker build -t catalogue:${env.appVersion} ."
                       docker images
                    """
                }
            }
        }

        stage('Deploy') {
            // input {
            //     message "Should we continue?"
            //     ok "Yes, we should."
            //     submitter "alice,bob"
            //     parameters {
            //         string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')
            //     }
            // }
            when { 
                expression { "$params.DEPLOY" == "true" }
            }
            steps {
                echo "Deploying"
            }
        }
    }

    post {
        always {
            echo 'I will always say Hello again!'
            cleanWs()
        }
        success {
            echo 'I will run if success'
        }
        failure {
            echo 'I will run if failure'
        }
         aborted {
            echo 'pipeline is aborted'
        }
    }
}