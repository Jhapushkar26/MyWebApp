pipeline {
    agent any

    environment {
        GITHUB_REPO = 'Jhapushkar26/MyWebApp'
        GITHUB_USER = 'Jhapushkar26'
        GITHUB_TOKEN = credentials('github-token5')
        REPO = 'Jhapushkar26/MyWebApp'
        BASE_BRANCH = 'main'
        CODECLIMATE_TEST_REPORTER_ID = credentials('code-climate-reporter-id')  // Stored in Jenkins Credentials
    }

    stages {

        stage('Install Dependencies') {
            steps {
                script {
                    echo "📦 Installing Required Linters"
                    sh '''
                    npm install -g htmlhint stylelint eslint
                    '''
                }
            }
        }


        stage('Checkout Code') {
            steps {
                script {
                    def branchName = env.BRANCH_NAME ?: 'main'  // Dynamically detect branch
                    echo "🔍 Checking out branch: ${branchName}"
                    git branch: branchName, url: "https://github.com/${GITHUB_REPO}"
                }
            }
        }

        stage('Verify Branch') {
            steps {
                script {
                    sh '''
                    echo "Current Git Branch: $(git rev-parse --abbrev-ref HEAD)"
                    git status
                    '''
                }
            }
        }

        stage('Code Climate Analysis') {
            steps {
                script {
                    sh '''
                    echo "🚀 Running Code Climate Analysis"
                    docker run --rm \
                        -v "$(pwd)":/code \
                        -w /code \
                        codeclimate/codeclimate analyze --debug
                    '''
                }
            }
        }

        stage('Code Quality Check') {
    steps {
        script {
            def result = sh(script: '''
            echo "📢 Running Code Climate Analysis for HTML with JSON output"
            docker run --rm \
                -v "$(pwd)":/code \
                -w /code \
                codeclimate/codeclimate analyze index.html --json | tee codeclimate-report.json
            ''', returnStdout: true).trim()

            if (result.contains('"severity": "critical"') || result.contains('"severity": "major"')) {
                error "❌ Code Climate found critical or major HTML issues! Fix them before proceeding."
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
                    echo "📢 Formatting and Uploading Coverage Report"
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
                withCredentials([string(credentialsId: 'github-token5', variable: 'GITHUB_TOKEN')]) {
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
                echo "📢 Deploying to Development VM"
                net use \\\\192.168.56.102\\wwwroot Pushkar123$ /USER:jenkinsuser
                xcopy /E /I /Y * \\\\192.168.56.102\\wwwroot\\
                net use /delete \\\\192.168.56.102\\wwwroot
                '''
            }
        }

    }
}
