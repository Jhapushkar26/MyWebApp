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
                withCredentials([string(credentialsId: 'github-token4', variable: 'GITHUB_TOKEN')]) {
                    script {
                        def branchName = env.GIT_BRANCH.replaceFirst('origin/', '')
                        if (!branchName) {
                            error "❌ ERROR: Branch name not detected!"
                        }

                        echo "ℹ️ Branch Name: ${branchName}"
                        echo "🔍 GitHub Repo: ${GITHUB_REPO}"
                        echo "📢 Base Branch: ${BASE_BRANCH}"

                        def prResponse = sh(script: """
                        response=\$(curl -s -w "\\nHTTP_STATUS:%{http_code}" -X POST \\
                            -H "Authorization: Bearer \$GITHUB_TOKEN" \\
                            -H "Accept: application/vnd.github.v3+json" \\
                            -d '{"title": "Automated PR from '"\$branchName"'","head": "'"\$branchName"'","base": "'"$BASE_BRANCH"'","body": "This is an automated PR created by Jenkins."}' \\
                            "https://api.github.com/repos/\$GITHUB_REPO/pulls")

                        http_status=\$(echo "\$response" | grep "HTTP_STATUS" | awk -F: '{print \$2}' | tr -d ' ')

                        if [ "\$http_status" -eq 422 ]; then
                            echo "⚠️ A PR already exists for this branch. Skipping..."
                        elif [ "\$http_status" -ne 201 ]; then
                            echo "❌ ERROR: PR creation failed with status \$http_status"
                            exit 1
                        else
                            echo "✅ Pull request created successfully!"
                        fi
                        """, returnStdout: true).trim()
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
