pipeline {
agent any

```
environment {
    IMAGE_TAG = "${BUILD_NUMBER}"
}

stages {

    stage('Checkout') {
        steps {
            git credentialsId: 'f87a34a8-0e09-45e7-b9cf-6dc68feac670',
                url: 'https://github.com/iam-veeramalla/cicd-end-to-end',
                branch: 'main'
        }
    }

    stage('Build Docker') {
        steps {
            bat '''
            echo Build Docker Image
            docker build -t abhishekf5/cicd-e2e:%BUILD_NUMBER% .
            '''
        }
    }

    stage('Push the artifacts') {
        steps {
            bat '''
            echo Push Docker Image
            docker push abhishekf5/cicd-e2e:%BUILD_NUMBER%
            '''
        }
    }

    stage('Checkout K8S manifest SCM') {
        steps {
            git credentialsId: 'f87a34a8-0e09-45e7-b9cf-6dc68feac670',
                url: 'https://github.com/iam-veeramalla/cicd-demo-manifests-repo.git',
                branch: 'main'
        }
    }

    stage('Update K8S manifest & push to Repo') {
        steps {
            withCredentials([
                usernamePassword(
                    credentialsId: 'f87a34a8-0e09-45e7-b9cf-6dc68feac670',
                    usernameVariable: 'GIT_USERNAME',
                    passwordVariable: 'GIT_PASSWORD'
                )
            ]) {

                powershell '''
                Get-Content deploy.yaml

                (Get-Content deploy.yaml) `
                -replace "32","$env:BUILD_NUMBER" |
                Set-Content deploy.yaml

                Get-Content deploy.yaml
                '''

                bat '''
                git add deploy.yaml
                git commit -m "Updated deploy yaml | Jenkins Pipeline"
                git push https://%GIT_USERNAME%:%GIT_PASSWORD%@github.com/iam-veeramalla/cicd-demo-manifests-repo.git HEAD:main
                '''
            }
        }
    }
}
```

}

