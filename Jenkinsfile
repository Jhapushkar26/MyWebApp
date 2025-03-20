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
                    def jenkinsfileExists = sh(script: "git ls-remote origin ${branchName} | grep Jenkinsfile || echo 'missing'", returnStdout: true).trim()

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
                        response=$(curl -s -w "\\nHTTP_STATUS:%{http_code}" -X POST \
                            -H "Authorization: token $GITHUB_TOKEN" \
                            -H "Accept: application/vnd.github.v3+json" \
                            -d '{
                                "title": "Automated PR from '"$BRANCH_NAME"'",
                                "head": "'"$BRANCH_NAME"'",
                                "base": "main",
                                "body": "This is an automated PR created by Jenkins."
                            }' \
                            "https://api.github.com/repos/$REPO/pulls")

                        http_status=$(echo "$response" | grep "HTTP_STATUS" | awk -F: '{print $2}')
                        api_response=$(echo "$response" | sed -e 's/HTTP_STATUS:[0-9]*//')

                        echo "GitHub API Response: $api_response"
                        echo "HTTP Status Code: $http_status"

                        if [ "$http_status" -ne 201 ]; then
                            echo "Error: Failed to create PR!"
                            exit 1
                        fi
                        ''', returnStdout: true).trim()

                        echo "Final GitHub API Response: ${response}"
                    }
                }
            }
        }
    }
}
