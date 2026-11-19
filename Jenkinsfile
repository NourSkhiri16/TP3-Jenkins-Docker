

pipeline {
    agent any

    options {
        timestamps()
    }

    stages {
        // 1. BUILD
        stage('Build') {
            steps {
                bat 'docker version'
                bat 'docker build -t monapp:build-%BUILD_NUMBER% .'
            }
        }

        // 2. TEST
        stage('Test') {
            steps {
                bat '''
                    @echo off
                    docker rm -f monapp-test 2>NUL
                    docker run -d --name monapp-test -p 8083:80 monapp:build-%BUILD_NUMBER%
                    if errorlevel 1 exit /b 1

                    powershell -NoProfile -Command "Start-Sleep -Seconds 3"

                    curl.exe -f http://localhost:8083/
                    set TEST_RESULT=%ERRORLEVEL%

                    docker rm -f monapp-test
                    exit /b %TEST_RESULT%
                '''
            }
        }

        // 3. PUSH
        stage('Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    powershell '''
                        $env:DOCKER_PASS | docker login -u $env:DOCKER_USER --password-stdin

                        if ($LASTEXITCODE -ne 0) {
                            exit 1
                        }

                        docker tag "monapp:build-$env:BUILD_NUMBER" "$env:DOCKER_USER/monapp:latest"
                        if ($LASTEXITCODE -ne 0) { exit 1 }

                        docker push "$env:DOCKER_USER/monapp:latest"
                        if ($LASTEXITCODE -ne 0) { exit 1 }

                        docker logout
                    '''
                }
            }
        }

        // 4. DEPLOY
        stage('Deploy') {
            steps {
                bat '''
                    docker rm -f monapp-deploy 2>NUL
                    docker run -d --name monapp-deploy -p 8082:80 %DOCKERHUB_USER%/monapp:latest
                '''
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished.'
        }
    }
}
