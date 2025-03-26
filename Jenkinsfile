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

            // Navigate to the project directory
            sh "cd C:/Users/pushkar.jha/Documents/MyWebApp && htmlhint index.html --format=json > htmlhint-report.json"

            // Debugging: Check if the report is created
            sh "ls -l C:/Users/pushkar.jha/Documents/MyWebApp/htmlhint-report.json"
            sh "cat C:/Users/pushkar.jha/Documents/MyWebApp/htmlhint-report.json"

            // Checking for errors
            def htmlLintErrors = sh(script: "grep 'error' C:/Users/pushkar.jha/Documents/MyWebApp/htmlhint-report.json || true", returnStatus: true)

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
