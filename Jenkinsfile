
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
                    powershell -NoProfile -Command "Start-Sleep -Seconds 3"
                    curl.exe -f http://localhost:8083/
                    set TEST_RESULT=%ERRORLEVEL%
                    docker rm -f monapp-test
                    exit /b %TEST_RESULT%
                '''
            }
        }

        stage('Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    powershell '''
                        $psi = New-Object System.Diagnostics.ProcessStartInfo
                        $psi.FileName = "docker"
                        $psi.Arguments = "login --username $env:DOCKER_USER --password-stdin"
                        $psi.UseShellExecute = $false
                        $psi.RedirectStandardInput = $true
                        $psi.RedirectStandardOutput = $true
                        $psi.RedirectStandardError = $true

                        $p = New-Object System.Diagnostics.Process
                        $p.StartInfo = $psi
                        [void]$p.Start()
                        $p.StandardInput.WriteLine($env:DOCKER_PASS)
                        $p.StandardInput.Close()
                        $p.WaitForExit()

                        if ($p.ExitCode -ne 0) {
                            Write-Error $p.StandardError.ReadToEnd()
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
    }

    post {
        always {
            echo 'Pipeline finished.'
        }
    }
}
