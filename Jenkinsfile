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

            // Debugging: Display contents of the reports before checking errors
            echo "🔍 Checking HTML Lint Report:"
            sh "cat htmlhint-report.json || echo '⚠️ htmlhint-report.json not found!'"

            echo "🔍 Checking CSS Lint Report:"
            sh "cat stylelint-report.json || echo '⚠️ stylelint-report.json not found!'"

            echo "🔍 Checking JS Lint Report:"
            sh "cat eslint-report.json || echo '⚠️ eslint-report.json not found!'"

            // Check for linting errors and capture both exit status and output
            def htmlLintOutput = sh(script: "grep 'error' htmlhint-report.json || true", returnStdout: true).trim()
            def cssLintOutput = sh(script: "grep 'error' stylelint-report.json || true", returnStdout: true).trim()
            def jsLintOutput = sh(script: "grep 'error' eslint-report.json || true", returnStdout: true).trim()

            def htmlLintErrors = sh(script: "grep 'error' htmlhint-report.json || true", returnStatus: true)
            def cssLintErrors = sh(script: "grep 'error' stylelint-report.json || true", returnStatus: true)
            def jsLintErrors = sh(script: "grep 'error' eslint-report.json || true", returnStatus: true)

            // Debugging: Print captured outputs and exit statuses
            echo "📄 HTML Lint Errors Output: ${htmlLintOutput}"
            echo "📄 CSS Lint Errors Output: ${cssLintOutput}"
            echo "📄 JS Lint Errors Output: ${jsLintOutput}"
            
            echo "📌 HTML Lint Exit Status: ${htmlLintErrors}"
            echo "📌 CSS Lint Exit Status: ${cssLintErrors}"
            echo "📌 JS Lint Exit Status: ${jsLintErrors}"

            // If any linter reports an error, fail the build
            if (htmlLintErrors == 0 || cssLintErrors == 0 || jsLintErrors == 0) {
                error "❌ Linting errors found! Fix them before proceeding."
            } else {
                echo "✅ Code Linting Passed!"
            }
        }
    }
}

    }
}
