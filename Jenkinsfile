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
                        git fetch origin main
                        git show-ref --quiet --verify refs/heads/main && git checkout main -- Jenkinsfile || echo 'Jenkinsfile not found in main'
                        git add Jenkinsfile
                        git commit -m 'Auto-adding Jenkinsfile to ${branchName}' || echo 'No changes to commit'
                        git push origin ${branchName}
                        """
                    }
                }
            }
        }
        
        stage('Create Pull Request') {
            steps {
                script {
                    def branchName = env.BRANCH_NAME
                    
                    sh """
                    curl -X POST -H "Authorization: token $GITHUB_TOKEN" \\
                         -H "Accept: application/vnd.github.v3+json" \\
                         -d '{
                               "title": "Automated PR from ${branchName}",
                               "head": "${branchName}",
                               "base": "main",
                               "body": "This is an automated PR created by Jenkins."
                             }' \\
                         https://api.github.com/repos/$REPO/pulls
                    """
                }
            }
        }
    }
}
