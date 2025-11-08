pipeline {
    agent any
    triggers {
        pollSCM('H/5 * * * *') // vérifie les changements toutes les 5 minutes
    }
    environment {
        IMAGE_SERVER = 'AmiraElf/mern-server'   // Image Docker serveur
        IMAGE_CLIENT = 'AmiraElf/mern-client'   // Image Docker client
    }
    stages {
        stage('Checkout') {
            steps {
                // Clonage du repo GitHub avec SSH
                git branch: 'main',
                    url: 'git@github.com:AmiraElf/TP2.git',
                    credentialsId: 'github_ssh'
            }
        }
        stage('Build + Push SERVER') {
            when { changeset 'server/**' }  // ne build que si dossier server modifié
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub',  // Credential Docker Hub
                    usernameVariable: 'DH_USER',
                    passwordVariable: 'DH_PASS'
                )]) {
                    sh '''
                        echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin
                        docker build -t $IMAGE_SERVER:${BUILD_NUMBER} server
                        docker push $IMAGE_SERVER:${BUILD_NUMBER}
                    '''
                }
            }
        }
        stage('Build + Push CLIENT') {
            when { changeset 'client/**' }  // ne build que si dossier client modifié
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub',
                    usernameVariable: 'DH_USER',
                    passwordVariable: 'DH_PASS'
                )]) {
                    sh '''
                        echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin
                        docker build -t $IMAGE_CLIENT:${BUILD_NUMBER} client
                        docker push $IMAGE_CLIENT:${BUILD_NUMBER}
                    '''
                }
            }
        }
        stage('Scan Images with Trivy') {
            steps {
                sh '''
                    # Analyse des images avec Trivy
                    docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
                    aquasec/trivy image $IMAGE_SERVER:${BUILD_NUMBER} > trivy_server_report.txt || true
                    docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
                    aquasec/trivy image $IMAGE_CLIENT:${BUILD_NUMBER} > trivy_client_report.txt || true
                '''
            }
        }
    }
    post {
        always {
            // Nettoyage des images et containers inutiles
            sh 'docker system prune -af || true'
        }
    }
}
