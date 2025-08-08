pipeline {
    agent any
    environment {
        REPO_NAME = 'bhogendra/simplejavaapp' // lowercase and full repo name
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
    }
}
