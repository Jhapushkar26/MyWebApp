pipeline {
    agent any
    environment {
        REPO = 'Jhapushkar26/MyWebApp'
    }
    stages {
        stage('Ensure Jenkinsfile Exists') {
            steps {
                script {
                    def branchName = env.BRANCH_NAME
                    def jenkinsfileExists = sh(script: "git ls-remote --exit-code origin ${branchName} | grep Jenkinsfile || echo 'missing'", returnStdout: true).trim()

                    if (jenkinsfileExists == 'missing') {
                        sh """
                        git fetch origin
                        git checkout main -- Jenkinsfile
                        git add Jenkinsfile
                        git commit -m "Auto-adding Jenkinsfile to ${branchName}"
                        git push origin ${branchName}
                        """
                    }
                }
            }
        }
        stage('Create Pull Request') {
            steps {
                withCredentials([string(credentialsId: 'github-token4', variable: 'GITHUB_TOKEN')]) {
                    script {
                        def response = sh(script: '''
                        curl -X POST -H "Authorization: token $GITHUB_TOKEN" \
                            -H "Accept: application/vnd.github.v3+json" \
                            -d '{
                                "title": "Automated PR from '"$BRANCH_NAME"'",
                                "head": "'"$BRANCH_NAME"'",
                                "base": "main",
                                "body": "This is an automated PR created by Jenkins."
                            }' \
                            "https://api.github.com/repos/$REPO/pulls"
                        ''', returnStdout: true).trim()

                        echo "GitHub API Response: ${response}"
                    }
                }
            }
        }
    }
}
