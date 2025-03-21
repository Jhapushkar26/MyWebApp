pipeline {
    agent any

    environment {
        BASE_BRANCH = 'main'
        GITHUB_REPO = 'Jhapushkar26/MyWebApp'
        GITHUB_USER = 'Jhapushkar26'
        GITHUB_TOKEN = credentials('github-username-and-pat')
        SONARQUBE_URL = 'http://localhost:9000'
        SONARQUBE_CREDENTIALS = 'sonarqube-token-new'
        SONAR_TOKEN = credentials('sonarqube-token-new')
    }

    stages {
        

        stage('Checkout Code') {
            steps {
                git branch: BASE_BRANCH, url: "https://github.com/${GITHUB_REPO}"
            }
        }

        stage('SonarQube Analysis') {
            steps {
               withSonarQubeEnv('SonarQube') {
                    bat 'sonar-scanner -Dsonar.projectKey=MyProject -Dsonar.sources=. -Dsonar.host.url=http://localhost:9000'
                }
            }
        }

        stage('Quality Gate Check') {
            steps {
                script {
                    def qualityGate = waitForQualityGate()
                    if (qualityGate.status != 'OK') {
                        error "❌ Quality Gate failed! Fix issues before deploying."
                    } else {
                        echo "✅ Quality Gate passed! Proceeding with deployment."
                    }
                }
            }
        }

        stage('Create Pull Request') {
            steps {
                withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                    script {
                        if (!env.BRANCH_NAME) {
                            error "❌ ERROR: BRANCH_NAME is not set! Ensure this pipeline is triggered by a branch."
                        }

                        echo "ℹ️ Branch Name: ${env.BRANCH_NAME}"
                        echo "🔍 GitHub Repo: ${GITHUB_REPO}"
                        echo "📢 Base Branch: ${BASE_BRANCH}"

                        def response = sh(script: """
                        response=\$(curl -s -w "\nHTTP_STATUS:%{http_code}" -X POST \\
                            -H "Authorization: Bearer \$GITHUB_TOKEN" \\
                            -H "Accept: application/vnd.github.v3+json" \\
                            -d '{"title": "Automated PR from '"\$BRANCH_NAME"'","head": "'"\$BRANCH_NAME"'","base": "'"$BASE_BRANCH"'","body": "This is an automated PR created by Jenkins."}' \\
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
                bat "net use \\\\192.168.56.102\\wwwroot ${'Pushkar123$'} /USER:jenkinsuser && xcopy /E /I /Y * \\\\192.168.56.102\\wwwroot\\ && net use /delete \\\\192.168.56.102\\wwwroot"
            }
        }
    }
}
