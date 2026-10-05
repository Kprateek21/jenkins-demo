pipeline {
    agent any
    options { skipDefaultCheckout() }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                powershell '''
                    $ErrorActionPreference = 'Stop'
                    New-Item -ItemType Directory -Force -Path build | Out-Null
                    Copy-Item index.html build/index.html -Force
                '''
            }
        }

        stage('Test') {
            steps {
                powershell '''
                    $ErrorActionPreference = 'Stop'
                    if (-not (Select-String -Path build/index.html -Pattern 'Jenkins deployment v2' -Quiet)) {
                        throw 'Expected heading was missing.'
                    }
                '''
            }
        }

        stage('Deploy') {
            steps {
                powershell '''
                    $ErrorActionPreference = 'Stop'
                    $site = Join-Path $env:USERPROFILE 'Documents\\jenkins-site'
                    New-Item -ItemType Directory -Force -Path $site | Out-Null
                    Copy-Item build/index.html (Join-Path $site 'index.html') -Force
                '''
            }
        }

        stage('Verify live site') {
            steps {
                powershell '''
                    $ErrorActionPreference = 'Stop'
                    $response = Invoke-WebRequest 'http://127.0.0.1:8081/' -UseBasicParsing
                    if ($response.StatusCode -ne 200 -or $response.Content -notmatch 'Jenkins deployment v2') {
                        throw 'The live site did not show v2.'
                    }
                    Write-Host 'Verified Git version at http://127.0.0.1:8081/'
                '''
            }
        }
    }
}