pipeline {
    agent any

    environment {
        GITHUB_REPO = 'Jhapushkar26/MyWebApp'
        GITHUB_USER = 'Jhapushkar26'
        REPO = 'Jhapushkar26/MyWebApp'
        BASE_BRANCH = 'main'
    }

    stages {
        stage('Checkout Feature Branch') {
            steps {
                script {
                    if (!env.BRANCH_NAME) {
                        error "❌ ERROR: BRANCH_NAME is not set!"
                    }
                    echo "🔍 Checking out branch: ${env.BRANCH_NAME}"
                    git branch: env.BRANCH_NAME, url: "https://github.com/${GITHUB_REPO}"
                }
            }
        }

        stage('Trigger GitHub Actions for Code Quality Check') {
            steps {
                withCredentials([string(credentialsId: 'github-token5', variable: 'GITHUB_TOKEN')]) {
                    script {
                        echo "🚀 Triggering GitHub Actions for branch: ${env.BRANCH_NAME}"
                        sh """
                        curl -X POST -H "Authorization: Bearer $GITHUB_TOKEN" -H "Accept: application/vnd.github.v3+json" \
                        "https://api.github.com/repos/${GITHUB_REPO}/actions/workflows/code-quality.yml/dispatches" \
                        -d '{"ref":"${env.BRANCH_NAME}"}'
                        """
                    }
                }
            }
        }

        stage('Wait for GitHub Actions to Complete') {
    steps {
        script {
            echo "⏳ Waiting for GitHub Actions to complete..."

            def maxRetries = 10  // 🔄 Retry up to 10 times (2.5 min total)
            def retryDelay = 15  // ⏳ Wait 15 seconds between retries
            def status = ""
            
            withCredentials([string(credentialsId: 'github-token5', variable: 'GITHUB_TOKEN')]) {
                for (int i = 0; i < maxRetries; i++) {
                    status = sh(script: """
                        curl -s -H "Authorization: Bearer $GITHUB_TOKEN" \
                        "https://api.github.com/repos/${GITHUB_REPO}/actions/runs" | jq -r \
                        '[.workflow_runs[] | select(.head_branch=="${env.BRANCH_NAME"})] | first | .conclusion'
                    """, returnStdout: true).trim()

                    if (status == "success") {
                        echo "✅ GitHub Actions passed. Proceeding to create PR."
                        break
                    } else if (status == "failure") {
                        error "❌ GitHub Actions failed. Fix issues before creating PR."
                    } else {
                        echo "⏳ Workflow not completed yet. Retrying in ${retryDelay}s..."
                        sleep(time: retryDelay, unit: 'SECONDS')
                    }
                }
            }

            if (status != "success") {
                error "❌ GitHub Actions did not complete successfully after multiple retries."
            }
        }
    }
}



        stage('Create Pull Request') {
            steps {
                withCredentials([string(credentialsId: 'github-token5', variable: 'GITHUB_TOKEN')]) {
                    script {
                        echo "📢 Creating Pull Request from ${env.BRANCH_NAME} to ${BASE_BRANCH}"

                        def response = sh(script: """
                        curl -s -w "\\nHTTP_STATUS:%{http_code}" -X POST \
                        -H "Authorization: Bearer $GITHUB_TOKEN" \
                        -H "Accept: application/vnd.github.v3+json" \
                        -d '{
                            "title": "Automated PR from ${env.BRANCH_NAME}",
                            "head": "${env.BRANCH_NAME}",
                            "base": "${BASE_BRANCH}",
                            "body": "This PR was created automatically after passing quality checks."
                        }' \
                        "https://api.github.com/repos/${REPO}/pulls"
                        """, returnStdout: true).trim()

                        echo "📢 GitHub API Response: ${response}"
                    }
                }
            }
        }
    }
}
                                                                                                                                                                                                                            