import java.util.Base64

pipeline {
        agent any

        environment {
            GITHUB_REPO = 'Jhapushkar26/MyWebApp'
            GITHUB_USER = 'Jhapushkar26'
            GITHUB_TOKEN = credentials('github-username-and-pat')
            SONARQUBE_URL = 'http://localhost:9000'
            SONARQUBE_CREDENTIALS = 'sonarqube-token-new'
            SONAR_TOKEN = 'squ_c0676d830f3f77841a6c0caa86bde636ada40ceb'
        }

        stages {
            stage('Checkout Code') {
                steps {
                    git branch: 'main', url: "https://github.com/${GITHUB_REPO}"
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
            timeout(time: 5, unit: 'MINUTES') {
                try {
                    echo "🔍 Checking Quality Gate status..."

                    def qualityGateStatus = ''
                    def maxRetries = 2
                    def retryCount = 0

                    while (retryCount < maxRetries) {
                        echo "🛠 Attempt ${retryCount + 1}: Sending request to SonarQube..."
                        echo "🌐 SonarQube API URL: ${env.SONARQUBE_URL}/api/qualitygates/project_status?projectKey=MyProject"

                        // Encode Sonar token using Java's Base64 Encoder
                        def encodedAuth = Base64.getEncoder().encodeToString("${env.SONAR_TOKEN}:".getBytes("UTF-8"))

                        def response
                        try {
                            response = httpRequest(
                                acceptType: 'APPLICATION_JSON',
                                url: "${env.SONARQUBE_URL}/api/qualitygates/project_status?projectKey=MyProject",
                                customHeaders: [[name: 'Authorization', value: "Basic ${encodedAuth}"]],
                                httpMode: 'GET'
                            )

                            echo "✅ Response Code: ${response.status}"

                            if (response.status != 200) {
                                echo "❌ Unexpected Response Code: ${response.status}"
                                throw new Exception("SonarQube API returned non-200 response.")
                            }

                            echo "🔹 Raw Response Content: ${response.content}"

                        } catch (Exception httpError) {
                            echo "⚠️ HTTP Request Failed: ${httpError.getMessage()}"
                            if (retryCount < maxRetries - 1) {
                                echo "⏳ Retrying in 30 seconds..."
                                sleep(30)
                                retryCount++
                                continue
                            } else {
                                error "❌ SonarQube API request failed after ${maxRetries} attempts."
                            }
                        }

                        // Parse JSON response
                        def jsonResponse
                        try {
                            jsonResponse = readJSON(text: response.content)
                            qualityGateStatus = jsonResponse.projectStatus.status
                            echo "📌 Extracted Quality Gate Status: ${qualityGateStatus}"
                        } catch (Exception jsonError) {
                            error "⚠️ JSON Parsing Failed: ${jsonError.getMessage()}"
                        }

                        if (qualityGateStatus == 'OK') {
                            echo "✅ Quality Gate passed! Proceeding with deployment."
                            break
                        } else {
                            echo "❌ Quality Gate failed. Attempt ${retryCount + 1} of ${maxRetries}..."
                            if (retryCount < maxRetries - 1) {
                                echo "⏳ Retrying in 30 seconds..."
                                sleep(30)
                            }
                        }
                        retryCount++
                    }

                    if (qualityGateStatus != 'OK') {
                        error "❌ Quality Gate failed after ${maxRetries} attempts! Fix issues before deploying."
                    }
                } catch (Exception e) {
                    echo "⚠️ Unexpected Error: ${e.getMessage()}"
                    error "❌ Pipeline Failed: ${e}"
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

