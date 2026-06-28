stage('Skip Jenkins Commits') {
    steps {
        script {
            def commitMessage = bat(
                script: 'git log -1 --pretty=%B',
                returnStdout: true
            ).trim()

            if (commitMessage.contains('Updated deploy.yaml')) {
                currentBuild.result = 'NOT_BUILT'
                error('Skipping Jenkins-generated commit')
            }
        }
    }
}

pipeline {
    agent any

    environment {
        IMAGE_TAG = "${BUILD_NUMBER}"
        DOCKER_CONFIG = "C:\\ProgramData\\Jenkins\\.docker"
        DOCKER_BUILDKIT = "1"
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
        \$content = Get-Content deploy\\deploy.yaml
        \$content = \$content -replace '6380575356/cicd-e2e:[^\\s]+', '6380575356/cicd-e2e:${env.BUILD_NUMBER}'
        \$content | Set-Content deploy\\deploy.yaml
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
                    git remote set-url origin https://%GIT_USERNAME%:%GIT_PASSWORD%@github.com/venkadesht/cicd_venkadesh.git
                    git add deploy\\deploy.yaml
                    git commit -m "Updated deploy.yaml | Jenkins Build %BUILD_NUMBER%" || exit 0
                    git push origin HEAD:main
                    """
                }
            }
        }
    }
}
