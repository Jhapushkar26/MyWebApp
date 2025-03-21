pipeline {
    agent any

    environment {
        GITHUB_REPO = 'Jhapushkar26/MyWebApp'
        GITHUB_USER = 'Jhapushkar26'
        GITHUB_TOKEN = credentials('github-username-and-pat')
        SONARQUBE_URL = 'http://localhost:9000'
        SONARQUBE_CREDENTIALS = 'sonarqube-token-new'
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

        stage('Create Pull Request') {
            steps {
                script {
                    def branchName = "feature-branch"
                    sh "git checkout -b ${branchName}"
                    sh "git push origin ${branchName}"
                    sh "curl -u ${GITHUB_USER}:${GITHUB_TOKEN} -X POST -d '{"title": "Automated PR", "head": "${branchName}", "base": "main"}' https://api.github.com/repos/${GITHUB_REPO}/pulls"
                }
            }
        }

        stage('Deploy to Development VM') {
            steps {
                bat "net use \\\\192.168.56.102\\wwwroot ${'Pushkar123$'} /USER:jenkinsuser && xcopy /E /I /Y * \\\\192.168.56.102\\wwwroot\\ && net use /delete \\\\192.168.56.102\\wwwroot"
            }
        }
    }
}