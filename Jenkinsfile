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
                        bat '''
                        sonar-scanner -Dsonar.projectKey=MyProject -Dsonar.sources=. ^
                                      -Dsonar.host.url=http://localhost:9000 ^
                                      -Dsonar.token=%SONAR_TOKEN%
                        '''
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    timeout(time: 5, unit: 'MINUTES') {
                        def qg = waitForQualityGate abortPipeline: true, credentialsId: 'sonarqube-token-new'
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
                    }
                }
            }
        }

        stage('Create Pull Request') {
            steps {
                script {
                    def branchName = "feature-branch"
                    sh """
                        git checkout -b ${branchName}
                        git push origin ${branchName}
                        curl -u ${GITHUB_USER}:${GITHUB_TOKEN} -X POST -H "Content-Type: application/json" ^
                             -d '{\"title\": \"Automated PR\", \"head\": \"${branchName}\", \"base\": \"main\"}' ^
                             https://api.github.com/repos/${GITHUB_REPO}/pulls
                    """
                }
            }
        }

        stage('Deploy to Development VM') {
            steps {
                withCredentials([string(credentialsId: 'deploy-password', variable: 'DEPLOY_PASS')]) {
                    bat '''
                    net use \\\\192.168.56.102\\wwwroot %DEPLOY_PASS% /USER:jenkinsuser
                    xcopy /E /I /Y * \\\\192.168.56.102\\wwwroot\\
                    net use /delete \\\\192.168.56.102\\wwwroot
                    '''
                }
            }
        }
    }
}
