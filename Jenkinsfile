pipeline {
    agent any

    environment {
        GITHUB_REPO = 'Jhapushkar26/MyWebApp'
        GITHUB_USER = 'Jhapushkar26'
        GITHUB_TOKEN = credentials('github-username-and-pat')
        SONARQUBE_URL = 'http://localhost:9000'
        SONARQUBE_CREDENTIALS = 'sonarqube-token-new'
        SONAR_TOKEN = 'squ_c0676d830f3f77841a6c0caa86bde636ada40ceb'
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
                    bat 'sonar-scanner -Dsonar.projectKey=MyProject -Dsonar.sources=. -Dsonar.host.url=http://localhost:9000'
                }
            }
        }

        stage('Quality Gate') {
    steps {
        script {
            sleep(60)
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
                net use \\\\192.168.56.102\\wwwroot Pushkar123$ /USER:jenkinsuser
                xcopy /E /I /Y * \\\\192.168.56.102\\wwwroot\\
                net use /delete \\\\192.168.56.102\\wwwroot
                '''
            }
        }
    }
}
