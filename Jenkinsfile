pipeline {
    agent any

    environment {
        GITHUB_REPO = 'Jhapushkar26/MyWebApp'
        GITHUB_USER = 'Jhapushkar26'
        GITHUB_TOKEN = credentials('github-token5')
        REPO = 'Jhapushkar26/MyWebApp'
        BASE_BRANCH = 'main'
        CODECLIMATE_TEST_REPORTER_ID = credentials('code-climate-reporter-id')
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

        stage('Lint HTML Files') {
            steps {
                script {
                    echo "🔍 Running HTMLHint"
                    sh '''
                    htmlhint "**/*.html" --format=json > htmlhint-report.json || true
                    '''
                }
            }
        }

        stage('Lint CSS Files') {
            steps {
                script {
                    echo "🔍 Running Stylelint"
                    sh '''
                    stylelint "**/*.css" --formatter json > stylelint-report.json || true
                    '''
                }
            }
        }

        stage('Lint JavaScript Files') {
            steps {
                script {
                    echo "🔍 Running ESLint"
                    sh '''
                    eslint "**/*.js" -f json -o eslint-report.json || true
                    '''
                }
            }
        }

        stage('Code Quality Check') {
            steps {
                script {
                    echo "📢 Checking for Linter Errors"

                    // Ensure the working directory is correct
                    sh "pwd"
                    sh "ls -l"

                    // Run HTMLHint and log output
                    sh "htmlhint index.html --format=json > htmlhint-report.json || echo '⚠️ htmlhint failed!'"
                    sh "ls -l htmlhint-report.json || echo '⚠️ htmlhint-report.json not found!'"
                    sh "cat htmlhint-report.json || echo '⚠️ No output in htmlhint-report.json'"

                    // Check if errors exist
                    def htmlLintErrors = sh(script: "grep 'error' htmlhint-report.json || true", returnStatus: true)

                    if (htmlLintErrors == 0) {
                        error "❌ Linting errors found! Fix them before proceeding."
                    } else {
                        echo "✅ Code Linting Passed!"
                    }
                }
            }
}




    }
}
