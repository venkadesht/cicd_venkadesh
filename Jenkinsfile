pipeline {
    agent any

    environment {
        IMAGE_TAG = "${BUILD_NUMBER}"
        DOCKER_CONFIG = "C:\\ProgramData\\Jenkins\\.docker"
    }

    stages {

        stage('Checkout') {
            steps {
                git credentialsId: 'github-creds',
                    url: 'https://github.com/venkadesht/cicd_venkadesh.git',
                    branch: 'main'
            }
        }

        stage('Build Docker') {
            steps {
                bat """
                echo Building Docker Image
                docker build -t 6380575356/cicd-e2e:%BUILD_NUMBER% .
                """
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    bat """
                    docker login -u %DOCKER_USERNAME% -p %DOCKER_PASSWORD%
                    docker push 6380575356/cicd-e2e:%BUILD_NUMBER%
                    docker logout
                    """
                }
            }
        }

        stage('Update Kubernetes Manifest') {
            steps {
                powershell """
                (Get-Content deploy\\\\deploy.yaml) `
                -replace 'v1', \$env:BUILD_NUMBER |
                Set-Content deploy\\\\deploy.yaml

                Get-Content deploy\\\\deploy.yaml
                """
            }
        }

        stage('Push Changes to GitHub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'github-creds',
                        usernameVariable: 'GIT_USERNAME',
                        passwordVariable: 'GIT_PASSWORD'
                    )
                ]) {
                    bat """
                    git config user.email "jenkins@local"
                    git config user.name "Jenkins"

                    git add deploy\\deploy.yaml
                    git commit -m "Updated deploy.yaml | Jenkins Build %BUILD_NUMBER%"
                    git push https://%GIT_USERNAME%:%GIT_PASSWORD%@github.com/venkadesht/cicd_venkadesh.git HEAD:main
                    """
                }
            }
        }
    }
}