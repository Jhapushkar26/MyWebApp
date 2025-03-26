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

            sh "which htmlhint || echo '⚠️ htmlhint not found!'"
            sh "which stylelint || echo '⚠️ stylelint not found!'"
            sh "which eslint || echo '⚠️ eslint not found!'"


            // 🗑 Delete old reports to avoid stale results
            sh "rm -f htmlhint-report.json stylelint-report.json eslint-report.json"

            // 📊 Run linters and generate fresh reports
            sh "htmlhint index.html --config .htmlhintrc --format=json > htmlhint-report.json"
            sh "stylelint '*.css' --formatter json > stylelint-report.json"
            sh "eslint '*.js' -f json > eslint-report.json"

            // 📝 Print reports for debugging
            echo "📜 HTML Lint Report:"
            sh "cat htmlhint-report.json || echo '⚠️ No report generated!'"

            echo "📜 CSS Lint Report:"
            sh "cat stylelint-report.json || echo '⚠️ No report generated!'"

            echo "📜 JS Lint Report:"
            sh "cat eslint-report.json || echo '⚠️ No report generated!'"

            // ✅ Check for errors
            def htmlLintErrors = sh(script: "jq 'length' htmlhint-report.json", returnStdout: true).trim()
            def cssLintErrors = sh(script: "jq 'length' stylelint-report.json", returnStdout: true).trim()
            def jsLintErrors = sh(script: "jq 'length' eslint-report.json", returnStdout: true).trim()

            if (htmlLintErrors != "0" || cssLintErrors != "0" || jsLintErrors != "0") {
                error "❌ Linting errors found! Fix them before proceeding."
            } else {
                echo "✅ Code Linting Passed!"
            }
        }
    }
}



    }
}
