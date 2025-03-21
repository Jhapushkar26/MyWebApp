pipeline {
    agent any

    environment {
        BASE_BRANCH = 'main'
        GITHUB_REPO = 'Jhapushkar26/MyWebApp'
        GITHUB_USER = 'Jhapushkar26'
        GITHUB_TOKEN = credentials('github-username-and-pat')
        SONARQUBE_URL = 'http://localhost:9000'
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
            withCredentials([string(credentialsId: 'sonarqube-token-new', variable: 'SONAR_AUTH_TOKEN')]) {
                bat """
                sonar-scanner ^
                    -Dsonar.projectKey=MyProject ^
                    -Dsonar.sources=. ^
                    -Dsonar.host.url=${SONARQUBE_URL} ^
                    -Dsonar.login=%SONAR_AUTH_TOKEN%
                """
            }
        }
    }
}



        stage('Quality Gate Check') {
            steps {
                script {
                    timeout(time: 5, unit: 'MINUTES') {
                        def qualityGate = waitForQualityGate()
                        if (qualityGate.status != 'OK') {
                            error "❌ Quality Gate failed! Fix issues before deploying."
                        } else {
                            echo "✅ Quality Gate passed! Proceeding with deployment."
                        }
                    }
                }
            }
        }

        stage('Create Pull Request') {
            steps {
                withCredentials([string(credentialsId: 'github-username-and-pat', variable: 'GITHUB_TOKEN')]) {
                    script {
                        def branchName = env.GIT_BRANCH.replaceFirst('origin/', '')
                        if (!branchName) {
                            error "❌ ERROR: Branch name not detected!"
                        }

                        echo "ℹ️ Branch Name: ${branchName}"
                        echo "🔍 GitHub Repo: ${GITHUB_REPO}"
                        echo "📢 Base Branch: ${BASE_BRANCH}"

                        def prResponse = sh(script: """
                        response=\$(curl -s -w "\nHTTP_STATUS:%{http_code}" -X POST \\
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
