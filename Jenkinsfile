pipeline {
    agent any
    environment {
        REPO_NAME = 'bhogendra920/simplejavaapp' // lowercase and full repo name
    }

    stages {
        stage('Compile code') {
            steps {
                echo 'Packaging the app'
                sh 'mvn clean package'
            }
            post {
                success {
                    echo 'Now Archiving it...'
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }

        stage('Unit test') {
            steps {
                echo 'We are running unittest'
            }
        }

        stage('Build docker image') {
            steps {
                echo 'Building docker image'
                sh 'whoami'
                sh "docker image build -t ${env.REPO_NAME}:${BUILD_NUMBER} ."
            }
        }

        stage('Scan docker image') {
            steps {
                echo 'Scanning docker image'
                sh "trivy image ${env.REPO_NAME}:${BUILD_NUMBER}"
            }
        }

        stage('Push image to registry') {
            steps {
                echo 'Pushing images'
                withDockerRegistry([credentialsId: 'dockerhubcredentials', url: '']) {
                    sh "docker push ${env.REPO_NAME}:${BUILD_NUMBER}"
                }
            }
        }

        stage('Deploy to devenv') {
            steps {
                echo "Deploying to dev environment"
                sh '''
                    docker container stop mysimpleapp || true
                    docker container rm mysimpleapp || true
                    docker run -d --name mysimpleapp -p 8082:8080 ${REPO_NAME}:${BUILD_NUMBER}
                    docker ps --filter "name=mysimpleapp" --format "table {{.Names}}\t{{.Image}}\t{{.Ports}}"
                '''
            }
        }
    }

    post {
        success {
            emailext(
                subject: "✅ Jenkins Build SUCCESS: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                body: """Hello Team,

Good news! The Jenkins build succeeded.

Job: ${env.JOB_NAME}
Build Number: ${env.BUILD_NUMBER}
URL: ${env.BUILD_URL}

Regards,
Jenkins
""",
                to: 'your-email@example.com'
            )
        }
        failure {
            emailext(
                subject: "❌ Jenkins Build FAILURE: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                body: """Hello Team,

Unfortunately, the Jenkins build has failed.

Job: ${env.JOB_NAME}
Build Number: ${env.BUILD_NUMBER}
URL: ${env.BUILD_URL}

Please check the logs and take necessary action.

Regards,
Jenkins
""",
                to: 'your-email@example.com'
            )
        }
    }
}
