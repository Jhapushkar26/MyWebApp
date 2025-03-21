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
                checkout([$class: 'GitSCM', 
                    branches: [[name: "*/${BASE_BRANCH}"]],
                    userRemoteConfigs: [[
                        url: "https://github.com/${GITHUB_REPO}",
                        credentialsId: 'github-username-and-pat'
                    ]]
                ])
            }
        }

        stage('SonarQube Analysis') {
    steps {
        withSonarQubeEnv('SonarQube') {
            script {
                bat """
                sonar-scanner ^
                    -Dsonar.projectKey=MyProject ^
                    -Dsonar.sources=. ^
                    -Dsonar.host.url=${SONARQUBE_URL} ^
                    -Dsonar.login=${SONAR_TOKEN}
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
                        def branchName = env.GIT_BRANCH?.replaceFirst('origin/', '') ?: error("❌ ERROR: Branch name not detected!")
                        def prData = """{
                            "title": "Automated PR from ${branchName}",
                            "head": "${branchName}",
                            "base": "${BASE_BRANCH}",
                            "body": "This is an automated PR created by Jenkins."
                        }"""
                        
                        def response = sh(script: """
                        curl -s -w "\\nHTTP_STATUS:%{http_code}" -X POST \\
                            -H "Authorization: Bearer $GITHUB_TOKEN" \\
                            -H "Accept: application/vnd.github.v3+json" \\
                            -d '${prData}' \\
                            "https://api.github.com/repos/$GITHUB_REPO/pulls"
                        """, returnStdout: true).trim()

                        def httpStatus = response.split("HTTP_STATUS:")[1]?.trim()
                        
                        if (httpStatus == "422") {
                            echo "⚠️ A PR already exists for this branch. Skipping..."
                        } else if (httpStatus != "201") {
                            error "❌ ERROR: PR creation failed with status ${httpStatus}"
                        } else {
                            echo "✅ Pull request created successfully!"
                        }
                    }
                }
            }
        }

        stage('Deploy to Development VM') {
            steps {
                script {
                    def deployResult = bat(script: """
                    net use \\\\192.168.56.102\\wwwroot Pushkar123$ /USER:jenkinsuser
                    xcopy /E /I /Y * \\\\192.168.56.102\\wwwroot\\
                    net use /delete \\\\192.168.56.102\\wwwroot
                    """, returnStatus: true)

                    if (deployResult != 0) {
                        error "❌ Deployment failed!"
                    } else {
                        echo "✅ Deployment successful!"
                    }
                }
            }
        }
    }
}
