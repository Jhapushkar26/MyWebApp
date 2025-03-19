pipeline {
    agent any

    environment {
        GITHUB_REPO = "Jhapushkar26/MyWebApp"  // Replace with your GitHub repo
        GITHUB_TOKEN = credentials('github-username-and-pat') // Use a Jenkins credential ID for GitHub token
    }

    stages {
        stage('Detect New Branch') {
            steps {
                script {
                    def branchName = env.BRANCH_NAME
                    if (branchName != 'main') {
                        echo "New branch detected: ${branchName}"
                    } else {
                        echo "Skipping main branch"
                        currentBuild.result = 'ABORTED'
                        error("Exiting build: Main branch does not need PR")
                    }
                }
            }
        }

        stage('Push Branch to GitHub') {
            steps {
                script {
                    echo "Pushing new branch ${env.BRANCH_NAME} to GitHub"
                    sh """
                    git remote add github https://github.com/${GITHUB_REPO}.git
                    git push github ${env.BRANCH_NAME}
                    """
                }
            }
        }

        stage('Create Pull Request') {
            steps {
                script {
                    def branchName = env.BRANCH_NAME
                    def targetBranch = 'main'

                    echo "Creating Pull Request from ${branchName} to ${targetBranch} in GitHub"

                    def payload = """
                    {
                        "title": "Auto-created PR for ${branchName}",
                        "head": "${branchName}",
                        "base": "${targetBranch}",
                        "body": "This PR was automatically created by Jenkins"
                    }
                    """

                    sh """
                    curl -X POST -H "Authorization: token ${GITHUB_TOKEN}" \\
                        -H "Accept: application/vnd.github.v3+json" \\
                        -d '${payload}' \\
                        https://api.github.com/repos/${GITHUB_REPO}/pulls
                    """
                }
            }
        }
    }
}
