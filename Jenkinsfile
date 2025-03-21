pipeline {
    agent any

    environment {
        GITHUB_REPO = 'Jhapushkar26/MyWebApp'
        GITHUB_USER = 'Jhapushkar26'
        GITHUB_TOKEN = credentials('github-username-and-pat')
        SONARQUBE_URL = 'http://localhost:9000'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: "https://github.com/${GITHUB_REPO}"
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    withCredentials([string(credentialsId: 'sonarqube-token-new', variable: 'SONAR_TOKEN')]) {
                        bat """
                        sonar-scanner -Dsonar.projectKey=MyProject ^
                                      -Dsonar.sources=. ^
                                      -Dsonar.host.url=${SONARQUBE_URL} ^
                                      -Dsonar.login=%SONAR_TOKEN%
                        """
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    def qualityGate = waitForQualityGate()
                    if (qualityGate.status != 'OK') {
                        error "Quality Gate failed: ${qualityGate.status}"
                    }
                }
            }
        }

        stage('Deploy to Development VM') {
            steps {
                bat '''
                net use \\\\192.168.56.102\\wwwroot Pushkar123$ /USER:jenkinsuser
                xcopy /E /I /Y * \\\\192.168.56.102\\wwwroot\\
                net use /delete \\\\192.168.56.102\\wwwroot
                '''
            }
        }
    }
}
