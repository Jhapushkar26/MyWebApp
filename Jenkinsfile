pipeline {
    agent any
    environment {
        GITHUB_TOKEN = credentials('github-username-and-pat')
        REPO = 'Jhapushkar26/MyWebApp'
    }
    stages {
        stage('Ensure Jenkinsfile Exists') {
            steps {
                script {
                    def branchName = env.BRANCH_NAME
                    def jenkinsfileExists = sh(script: "git ls-remote --exit-code origin ${branchName}:Jenkinsfile || echo 'missing'", returnStdout: true).trim()
                    
                    if (jenkinsfileExists == 'missing') {
                        sh """
                        git checkout main -- Jenkinsfile
                        git add Jenkinsfile
                        git commit -m 'Auto-adding Jenkinsfile to ${branchName}'
                        git push origin ${branchName}
                        """
                    }
                }
            }
        }
    }
}
