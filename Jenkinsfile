pipeline {
    agent any
    environment {
        REPO = 'Jhapushkar26/MyWebApp'
        BRANCH_NAME = 'feature-new-branch4'  // Ensure branch is explicitly set
    }
    stages {
        stage('Ensure Jenkinsfile Exists') {
            steps {
                script {
                    def jenkinsfileExists = sh(script: "git ls-remote --heads origin ${BRANCH_NAME} | wc -l", returnStdout: true).trim()

                    if (jenkinsfileExists == '0') {
                        echo "⚠️ Jenkinsfile missing in branch ${BRANCH_NAME}. Adding it now..."
                        sh '''
                        git fetch origin
                        git checkout ${BRANCH_NAME}
                        git restore --source=main -- Jenkinsfile  # Modern approach instead of `checkout main -- Jenkinsfile`
                        git add Jenkinsfile
                        git commit -m "Auto-adding Jenkinsfile to ${BRANCH_NAME}"
                        git push origin ${BRANCH_NAME}
                        '''
                    } else {
                        echo "✅ Jenkinsfile already exists in branch ${BRANCH_NAME}. No action needed."
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

                        if [ "$http_status" -eq 422 ]; then
                            echo "⚠️ Pull request already exists for ${BRANCH_NAME}. Skipping PR creation."
                            exit 0
                        elif [ "$http_status" -ne 201 ]; then
                            echo "❌ Error: Failed to create PR!"
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
