pipeline {
    agent any
    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:22-alpine'
					reuseNode true
                }
            }
            steps {
				sh '''
                ls -la
                node --version
                npm --version
                npm ci
                npm run build
                '''
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
                npm install serve
                node_modules/.bin/serve -s build &
                npx playwright test --reporter=html
                '''
            }
        }
    }
    post {
    always {
        junit 'test-results/junit.xml'
    }
}
}
