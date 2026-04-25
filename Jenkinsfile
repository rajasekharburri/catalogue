pipeline {
    agent {
        node {
            label 'AGENT-1'
        }
    }

    environment {
        COURSE = "Jenkins"
        appVersion = ""
        ACC_ID = "672945439745"
        PROJECT = "roboshop"
        COMPONENT = "catalogue"
    }

    options {
        timeout(time: 10, unit: 'MINUTES') 
        disableConcurrentBuilds()
    }

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

        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }

        stage('Unit Test') {
            steps {
                sh "npm install"
            }
        }

        stage('Build Image') {
            steps {
                script {
                    withAWS(region:'us-east-1', credentials:'aws-creds') {
                        sh """
                            aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com
                            docker build -t ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion} .
                            docker push ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion}
                        """
                    }
                }
            }
        }

        stage('Dependabot Security Gate') {
            environment {
                GITHUB_OWNER = 'rajasekharburri'
                GITHUB_REPO  = 'catalogue'
                GITHUB_API   = 'https://api.github.com'
                GITHUB_TOKEN = credentials('GITHUB_TOKEN')
            }
            steps {
                script {
                    sh '''
                    echo "Fetching Dependabot alerts..."
                    response=$(curl -s \
                        -H "Authorization: token ${GITHUB_TOKEN}" \
                        -H "Accept: application/vnd.github+json" \
                        "${GITHUB_API}/repos/${GITHUB_OWNER}/${GITHUB_REPO}/dependabot/alerts?per_page=100")

                    high_critical_open_count=$(echo "${response}" | jq '[.[] 
                        | select(.state == "open" 
                        and (.security_advisory.severity == "high" 
                        or .security_advisory.severity == "critical"))] | length')

                    if [ "${high_critical_open_count}" -gt 0 ]; then
                        exit 1
                    fi
                    '''
                }
            }
        }

        // stage('Trivy Scan') {
        //     steps {
        //         script {
        //             sh """
        //                 trivy image \
        //                 --severity HIGH,CRITICAL,MEDIUM \
        //                 --exit-code 1 \
        //                 ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion}
        //             """
        //         }
        //     }
        // }
    }

    post {
        always {
            cleanWs()
        }
        success {
            echo 'SUCCESS'
        }
        failure {
            echo 'FAILED'
        }
    }
}