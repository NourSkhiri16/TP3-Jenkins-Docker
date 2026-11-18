pipeline {
    agent any

    options {
        timestamps()
    }

    stages {
        stage('Build') {
            steps {
                bat 'docker version'
                bat 'docker build -t monapp:build-%BUILD_NUMBER% .'
            }
        }

        stage('Test') {
            steps {
                bat '''
                    @echo off
                    docker rm -f monapp-test 2>NUL
                    docker run -d --name monapp-test -p 8083:80 monapp:build-%BUILD_NUMBER%
                    if errorlevel 1 exit /b 1
                    timeout /t 3 /nobreak >NUL
                    curl.exe -f http://localhost:8083/
                    set TEST_RESULT=%ERRORLEVEL%
                    docker rm -f monapp-test
                    exit /b %TEST_RESULT%
                '''
            }
        }
    }

    post {
        always {
            bat 'docker rm -f monapp-test 2>NUL'
        }
    }
}
