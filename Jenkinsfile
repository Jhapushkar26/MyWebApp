pipeline {
    agent any
    environment {
        REPO = 'Jhapushkar26/MyWebApp'
        BASE_BRANCH = 'main'
        SONAR_PROJECT_KEY = 'sonarqube-token-new' 
        SONAR_HOST_URL = 'http://your-sonarqube-server:9000'
    }
    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        mvn clean verify sonar:sonar \
                        -Dsonar.projectKey=$SONAR_PROJECT_KEY \
                        -Dsonar.host.url=$SONAR_HOST_URL \
                        -Dsonar.login=$SONAR_TOKEN
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    timeout(time: 5, unit: 'MINUTES') {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "❌ Code quality check failed. Fix issues before proceeding!"
                        }
                    }
                }
            }
        }

        stage('Create Pull Request') {
            steps {
                withCredentials([string(credentialsId: 'github_token', variable: 'GITHUB_TOKEN')]) {
                    script {
                        echo "🔍 Running PR Creation for branch: ${env.BRANCH_NAME}"
                        
                        def response = sh(script: """
                        response=\$(curl -s -w "\\nHTTP_STATUS:%{http_code}" -X POST \\ 
                            -H 'Authorization: token \$GITHUB_TOKEN' \\ 
                            -H 'Accept: application/vnd.github.v3+json' \\ 
                            -d '{
                                "title": "Automated PR from '"\$BRANCH_NAME"'",
                                "head": "'"\$BRANCH_NAME"'",
                                "base": "'"\$BASE_BRANCH"'",
                                "body": "This PR is auto-generated after passing SonarQube checks."
                            }' \\ 
                            "https://api.github.com/repos/\$REPO/pulls")

                        http_status=\$(echo "\$response" | grep "HTTP_STATUS" | awk -F: '{print \$2}' | tr -d ' ')
                        api_response=\$(echo "\$response" | sed -e 's/HTTP_STATUS:[0-9]*//')

                        if [ "\$http_status" -eq 422 ]; then
                            echo "⚠️ A PR already exists for this branch. Skipping..."
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
    }
}
