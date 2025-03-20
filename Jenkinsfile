pipeline {
    agent any
    environment {
        GITHUB_TOKEN = credentials('github-username-and-pat') // Store GitHub token in Jenkins credentials
        REPO = 'Jhapushkar26/MyWebApp' // Your repository name
    }
    stages {
        stage('Create Pull Request') {
            steps {
                script {
                    def branchName = env.BRANCH_NAME
                    if (branchName != 'main' && branchName != 'master') { // Ensure it's not main
                        sh """
                        gh auth login --with-token <<< \$GITHUB_TOKEN
                        gh pr create --base main --head ${branchName} --title 'New PR from ${branchName}' --body 'Automated PR creation'
                        """
                    }
                }
            }
        }
    }
}
