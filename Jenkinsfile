pipeline {
    agent any

    triggers {
        githubPush()
    }

    environment {
        IMAGE_REPO = 'gestion-projets-backend'
        IMAGE_TAG  = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Récupération du code') {
            steps { checkout scm }
        }

        stage('Tests unitaires') {
            steps {
                dir('backend') { sh 'mvn clean test' }
            }
            post {
                always {
                    junit allowEmptyResults: true, testResults: 'backend/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Création du livrable') {
            steps {
                dir('backend') { sh 'mvn package -DskipTests' }
            }
            post {
                success {
                    archiveArtifacts artifacts: 'backend/target/*.jar', fingerprint: true
                }
            }
        }

        stage("Build de l'image Docker") {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                                  usernameVariable: 'DOCKER_USER',
                                                  passwordVariable: 'DOCKER_PASS')]) {
                    dir('backend') {
                        sh 'docker build -t $DOCKER_USER/$IMAGE_REPO:$IMAGE_TAG -t $DOCKER_USER/$IMAGE_REPO:latest .'
                    }
                }
            }
        }

        stage('Push sur Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                                  usernameVariable: 'DOCKER_USER',
                                                  passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $DOCKER_USER/$IMAGE_REPO:$IMAGE_TAG
                        docker push $DOCKER_USER/$IMAGE_REPO:latest
                    '''
                }
            }
        }
    }

    post {
        always {
            sh 'docker logout || true'
        }
        failure {
            emailext(
                to: 'raif.guizani@esprit.tn',
                subject: "ECHEC - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """Le build ${env.JOB_NAME} #${env.BUILD_NUMBER} a échoué.
Consulter les logs : ${env.BUILD_URL}console"""
            )
        }
    }
}
