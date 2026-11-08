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
                sh 'echo "Building..."'
				sh '''
                ls -la
                node --version
                npm --version
                npm ci
                npm run build
                '''
            }
        }
    }
}
