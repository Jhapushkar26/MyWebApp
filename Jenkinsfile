pipeline {
    agent any

    environment {
        GITHUB_REPO = 'Jhapushkar26/MyWebApp'
        GITHUB_USER = 'Jhapushkar26'
        GITHUB_TOKEN = credentials('github-username-and-pat')
        SONARQUBE_URL = 'http://localhost:9000'
        SONARQUBE_CREDENTIALS = credentials('sonarqube-token-new') // Securely fetch token
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: "https://github.com/${GITHUB_REPO}"
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') { // Ensure this matches Jenkins config
                    bat '''
                    @echo off
                    sonar-scanner -Dsonar.projectKey=MyProject ^
                                  -Dsonar.sources=. ^
                                  -Dsonar.host.url=%SONARQUBE_URL% ^
                                  -Dsonar.login=%SONARQUBE_CREDENTIALS% ^
                                  -X
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    sleep(60) // Wait for analysis completion
                    timeout(time: 5, unit: 'MINUTES') {
                        def qualityGate = waitForQualityGate()
                        echo "Quality Gate Status: ${qualityGate.status}"
                        if (qualityGate.status != 'OK') {
                            error "Quality Gate failed: ${qualityGate.status}"
                        }
                    }
                }
            }
        }

        stage('Deploy to Development VM') {
            steps {
                bat '''
                @echo off
                echo Mapping network drive...
                net use \\\\192.168.56.102\\wwwroot Pushkar123$ /USER:jenkinsuser

                echo Deploying files...
                robocopy . \\\\192.168.56.102\\wwwroot /E /PURGE

                echo Disconnecting network drive...
                net use /delete \\\\192.168.56.102\\wwwroot
                '''
            }
        }
    }
}
