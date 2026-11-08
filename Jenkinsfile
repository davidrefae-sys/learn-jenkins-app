pipeline {
    agent any
    stages {

        stage ("Run test") {
            parallel {
                stage('Build') {
                    agent {
                        docker {
                            image 'node:22-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                        npm test
                        '''
                    }
                    post {
                    always {
                        junit allowEmptyResults: true, testResults: 'playwright-results/junit.xml'
                    }
                }
                }
                stage("E2E"){
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                            // args '-u root:root' // run as root, horrible solution, never do this
                        }
                    }
                    steps {
                        echo "Test Stage"
                        sh '''
                        npm ci
                        npm run build
                        npm install serve
                        node_modules/.bin/serve -s build &
                        PLAYWRIGHT_JUNIT_OUTPUT_NAME=playwright-results/junit.xml npx playwright test --reporter=junit
                        '''
                    }
                    post {
                    always {
                        publishHTML([
                            allowMissing: true,
                            alwaysLinkToLastBuild: false,
                            keepAll: true,
                            reportDir: 'playwright-report',
                            reportFiles: 'index.html',
                            reportName: 'Playwright HTML Report',
                            reportTitles: '',
                            useWrapperFileDirectly: true
                        ])
                    }
                }
                }
            }
        }

    }
}
