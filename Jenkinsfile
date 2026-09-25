pipeline {
    agent any

    // Déclenché par le webhook GitHub à chaque push
    triggers {
        githubPush()
    }

    stages {
        stage('Récupération du code') {
            steps {
                checkout scm
            }
        }

        stage('Tests unitaires') {
            steps {
                dir('backend') {
                    sh 'mvn clean test'
                }
            }
            post {
                always {
                    junit allowEmptyResults: true, testResults: 'backend/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Création du livrable') {
            steps {
                dir('backend') {
                    sh 'mvn package -DskipTests'
                }
            }
            post {
                success {
                    archiveArtifacts artifacts: 'backend/target/*.jar', fingerprint: true
                }
            }
        }
    }

    post {
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
