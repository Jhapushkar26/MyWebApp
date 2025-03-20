pipeline {
    agent any

    environment {
        GITHUB_TOKEN = credentials('github-token')  // Store GitHub Token in Jenkins Credentials
        GITHUB_REPO = 'Jhapushkar26/MyWebApp'      // Change this to your repo
        TARGET_BRANCH = 'main'                     // Change to your main branch name
    }

    stages {
        stage('Detect New Branch') {
            steps {
                script {
                    def branchName = env.BRANCH_NAME
                    if (branchName != TARGET_BRANCH) {
                        echo "New branch detected: ${branchName}"
                        createPullRequest(branchName)
                    } else {
                        echo "Not creating PR for the main branch."
                    }
                }
            }
        }
    }
}

def createPullRequest(String branchName) {
    def prTitle = "Auto PR: Merging ${branchName} into ${env.TARGET_BRANCH}"
    def prBody = "This PR was automatically created by Jenkins."

    def response = sh(script: """
        curl -X POST -H "Authorization: token ${env.GITHUB_TOKEN}" \
        -H "Accept: application/vnd.github.v3+json" \
        https://api.github.com/repos/${env.GITHUB_REPO}/pulls \
        -d '{
            "title": "${prTitle}",
            "head": "${branchName}",
            "base": "${env.TARGET_BRANCH}",
            "body": "${prBody}"
        }'
    """, returnStdout: true).trim()

    echo "GitHub API Response: ${response}"
}
