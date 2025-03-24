pipeline {
    agent any

    environment {
        GITHUB_REPO = 'Jhapushkar26/MyWebApp'
        GITHUB_USER = 'Jhapushkar26'
        GITHUB_TOKEN = credentials('github-token4')
        CODECLIMATE_TEST_REPORTER_ID = credentials('code-climate-reporter-id')  // Stored in Jenkins Credentials
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: "https://github.com/${GITHUB_REPO}"
            }
        }

        stage('Code Climate Analysis') {
            steps {
                script {
                    sh '''
                    docker run --rm \
                        -v "$(pwd)":/code \
                        -w /code \
                        codeclimate/codeclimate analyze
                    '''
                }
            }
        }

        stage('Code Quality Check') {
            steps {
                script {
                    def result = sh(script: '''
                    docker run --rm \
                        -v "$(pwd)":/code \
                        -w /code \
                        codeclimate/codeclimate analyze --json | tee codeclimate-report.json
                    ''', returnStdout: true).trim()

                    if (result.contains('"severity": "critical"')) {
                        error "❌ Code Climate found critical issues! Fix them before proceeding."
                    } else {
                        echo "✅ Code Climate analysis passed!"
                    }
                }
            }
        }

        stage('Upload Coverage Report to Code Climate') {
            steps {
                script {
                    sh '''
                    docker run --rm \
                        -e CODECLIMATE_TEST_REPORTER_ID=${CODECLIMATE_TEST_REPORTER_ID} \
                        -v "$(pwd)":/code \
                        -w /code \
                        codeclimate/codeclimate format-coverage coverage/lcov.info --output coverage/codeclimate.json

                    docker run --rm \
                        -e CODECLIMATE_TEST_REPORTER_ID=${CODECLIMATE_TEST_REPORTER_ID} \
                        -v "$(pwd)":/code \
                        -w /code \
                        codeclimate/codeclimate upload-coverage
                    '''
                }
            }
        }

        stage('Create Pull Request') {
            steps {
                withCredentials([string(credentialsId: 'github-token-id', variable: 'GITHUB_TOKEN')]) {
                    script {
                        if (!env.BRANCH_NAME) {
                            error "❌ ERROR: BRANCH_NAME is not set! Ensure this pipeline is triggered by a branch."
                        }

                        echo "ℹ️ Branch Name: ${env.BRANCH_NAME}"
                        echo "🔍 GitHub Repo: ${REPO}"
                        echo "📢 Base Branch: ${BASE_BRANCH}"

                        def response = sh(script: """
                        echo "✅ Checking Environment Variables"
                        echo "BRANCH_NAME: \$BRANCH_NAME"
                        echo "BASE_BRANCH: \$BASE_BRANCH"
                        echo "REPO: \$REPO"

                        echo "📢 Sending PR creation request to GitHub API"

                        response=\$(curl -s -w "\\nHTTP_STATUS:%{http_code}" -X POST \\
                            -H "Authorization: Bearer \$GITHUB_TOKEN" \\
                            -H "Accept: application/vnd.github.v3+json" \\
                            -d '{
                                "title": "Automated PR from '"\$BRANCH_NAME"'",
                                "head": "'"\$BRANCH_NAME"'",
                                "base": "'"\$BASE_BRANCH"'",
                                "body": "This is an automated PR created by Jenkins."
                            }' \\
                            "https://api.github.com/repos/\$REPO/pulls")

                        http_status=\$(echo "\$response" | grep "HTTP_STATUS" | awk -F: '{print \$2}' | tr -d ' ')
                        api_response=\$(echo "\$response" | sed -e 's/HTTP_STATUS:[0-9]*//')

                        echo "🔍 Full API Response: \$api_response"
                        echo "ℹ️ HTTP Status: \$http_status"

                        if [ "\$http_status" -eq 422 ]; then
                            echo "⚠️ A PR already exists for this branch. Skipping..."
                            exit 0
                        elif [ "\$http_status" -ne 201 ]; then
                            echo "❌ ERROR: PR creation failed with status \$http_status"
                            echo "🔍 API Response: \$api_response"
                            exit 1
                        else
                            echo "✅ Pull request created successfully!"
                        fi
                        """, returnStdout: true).trim()

                        echo "📢 Final GitHub API Response: ${response}"
                    }
                }
            }
        }
    

        stage('Deploy to Development VM') {
            steps {
                bat '''
                net use \\\\192.168.56.102\\wwwroot Pushkar123$ /USER:jenkinsuser
                xcopy /E /I /Y * \\\\192.168.56.102\\wwwroot\\
                net use /delete \\\\192.168.56.102\\wwwroot
                '''
            }
        }
    }
}
