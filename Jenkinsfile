pipeline {
    agent any
    environment {
        REPO = 'Jhapushkar26/MyWebApp'
        BASE_BRANCH = 'main'
    }
    stages {
        stage('Create Pull Request') {
            steps {
                withCredentials([string(credentialsId: 'github-token4', variable: 'GITHUB_TOKEN')]) {
                    script {
                        if (!env.BRANCH_NAME) {
                            error "❌ ERROR: BRANCH_NAME is not set! Ensure this pipeline is triggered by a branch."
                        }

                        echo "ℹ️ Creating PR for branch: ${env.BRANCH_NAME}"

                        def response = sh(script: """
                        #!/bin/bash
                        set -e  # Exit on error
                        set -x  # Enable debugging

                        echo "ℹ️ Checking if a pull request already exists for branch: $BRANCH_NAME"

                        response=\$(curl -s -w "\\nHTTP_STATUS:%{http_code}" -X POST \\
                            -H "Authorization: token $GITHUB_TOKEN" \\
                            -H "Accept: application/vnd.github.v3+json" \\
                            -d '{
                                "title": "Automated PR from '"$BRANCH_NAME"'",
                                "head": "'"$BRANCH_NAME"'",
                                "base": "'"$BASE_BRANCH"'",
                                "body": "This is an automated PR created by Jenkins."
                            }' \\
                            "https://api.github.com/repos/$REPO/pulls")

                        http_status=\$(echo "\$response" | grep "HTTP_STATUS" | awk -F: '{print \$2}' | tr -d ' ')
                        api_response=\$(echo "\$response" | sed -e 's/HTTP_STATUS:[0-9]*//')

                        echo "🔍 Full GitHub API Response:"
                        echo "\$api_response"

                        echo "ℹ️ HTTP Status Code: \$http_status"

                        if [ "\$http_status" -eq 422 ]; then
                            echo "⚠️ A pull request already exists for this branch. Skipping PR creation."
                            exit 0
                        elif [ "\$http_status" -ne 201 ]; then
                            echo "❌ ERROR: Failed to create PR! HTTP Status: \$http_status"
                            exit 1
                        else
                            echo "✅ Pull request created successfully!"
                        fi
                        """, returnStdout: true).trim()

                        echo "📢 Final GitHub API Response:\n${response}"
                    }
                }
            }
        }
    }
}
