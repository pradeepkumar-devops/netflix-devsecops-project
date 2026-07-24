pipeline {
    agent any

    environment {
        IMAGE_NAME = "pradeepkumarcr26/netflix-devsecops:v1"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git(
                    branch: 'main',
                    url: 'https://github.com/pradeepkumar-devops/netflix-devsecops-project.git'
                )
            }
        }

        stage('SonarQube Scan') {
            steps {
                script {
                    def scannerHome = tool 'sonar-scanner'
                    withSonarQubeEnv('SonarQube') {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=Netflix-DevSecOps-Project \
                            -Dsonar.sources=. \
                            -Dsonar.sourceEncoding=UTF-8
                        """
                    }
                }
            }
        }

        stage('Trivy Filesystem Scan') {
            steps {
                sh '''
                    mkdir -p $HOME/trivy-tmp
                    mkdir -p $HOME/.cache/trivy

                    export TMPDIR=$HOME/trivy-tmp
                    export TRIVY_CACHE_DIR=$HOME/.cache/trivy

                    trivy fs .
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME .'
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh '''
                    mkdir -p $HOME/trivy-tmp
                    mkdir -p $HOME/.cache/trivy

                    export TMPDIR=$HOME/trivy-tmp
                    export TRIVY_CACHE_DIR=$HOME/.cache/trivy

                    trivy image --scanners vuln $IMAGE_NAME
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $IMAGE_NAME
                    '''
                }
            }
        }

        stage('Deploy Container') {
            steps {
                sh 'docker rm -f netflix-container || true'
                sh 'docker run -d --name netflix-container -p 80:80 $IMAGE_NAME'
            }
        }
    }
}
