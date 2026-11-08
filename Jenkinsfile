- Can be installed using docker
- Can be easily installed using war file
- freestyle Job allows you to click through options and build with building blocks
- Pipeline job is configured by script : https://www.jenkins.io/doc/book/pipeline/syntax/
```groovy
pipeline {
    agent any

    stages {
        stage('Hello') {
            steps {
                echo 'Hello World'
                sh 'echo "hi from david"'
                sh "ls /"
            }
        }
    }
}
```
- Pipeline is essential a list of stages ( jobs)
- jenkins controller(stores results, manages the pipelines and the associated jobs) => jenkins agent (executes the jobs)
```groovy
pipeline {
    agent any

    stages {
        stage('Hello') {
            steps {
                echo 'Hello World'
                sh 'echo "mkdir -p build'
                sh "touch build/computer.txt"
				sh 'echo "Mainboard" >> build/computer.txt'
				sh 'cat build/computer.txt'
            }
        }
    }
}
```
- agents have workspace , you can see their contents from jenkins, you can cleanup post work
```groovy
pipeline {
    agent any

    stages {
        stage('Hello') {
            steps {
                echo 'Hello World'
                sh 'echo "mkdir -p build'
                sh "touch build/computer.txt"
				sh 'echo "Mainboard" >> build/computer.txt'
				sh 'cat build/computer.txt'
            }
        }
    }
  post {
	  always {
			cleanWs()
	  }
  }
}
```
- But we need to save some artifacts for later
```groovy
pipeline {
    agent any

    stages {
        stage('Hello') {
            steps {
				cleanWs()
                echo 'Hello World'
                sh 'echo "mkdir -p build'
                sh "touch build/computer.txt"
				sh 'echo "Mainboard" >> build/computer.txt'
				sh 'cat build/computer.txt'
            }
        }
    }
  post {
	  success {
	  	archiveArtifacts artifacts: "build/**" #always was removed because it has higher priority and will delete the files before they are archive
	  }
  }
}
```
- You can execute multiple commands at once using sh
```groovy
pipeline {
    agent any

    stages {
        stage('Hello') {
            steps {
                cleanWs()
                echo 'Hello World'
				sh '''
				echo "mkdir -p build"
	            mkdir -p build
			    touch build/computer.txt
                echo "Mainboard" >> build/computer.txt
  			    cat build/computer.txt
	            '''
            }
        }
    }
  post {
	  success {
	  	archiveArtifacts artifacts: "build/**"
	  }
  }
}
```
- You can also add another stage
```groovy
pipeline {
    agent any

    stages {
        stage('Hello') {
            steps {
                cleanWs()
                echo 'Hello World'
				sh '''
				echo "mkdir -p build"
	            mkdir -p build
			    touch build/computer.txt
                echo "Mainboard" >> build/computer.txt
  			    cat build/computer.txt
	            '''
            }
        }
        stage('Testing') { 
            steps {
            echo "testing the new hardware"
            sh '''
            test -f build/computer.txt
            grep -nr "Mainboard" build/computer.txt 
            '''
            }
        }
    }
  post {
	  success {
	  	archiveArtifacts artifacts: "build/**"
	  }
  }
}
```
- as we know its better to store re-used variables in an enviroment variable, keep in mind to use doible quotes when using the variable
```groovy
pipeline {
    agent any
    
    environment {
        BUILD_FILE_NAME = "laptop.txt"
    }
    stages {
        stage('Hello') {
            steps {
                cleanWs()
                echo 'Hello World'
                echo "$BUILD_FILE_NAME"
				sh '''
				echo "saving in file $BUILD_FILE_NAME"
				echo "mkdir -p build"
	            mkdir -p build
			    touch build/$BUILD_FILE_NAME
                echo "Mainboard" >> build/$BUILD_FILE_NAME
  			    cat build/$BUILD_FILE_NAME
                echo "Display" >> build/$BUILD_FILE_NAME
  			    cat build/$BUILD_FILE_NAME
  			    echo "Keyboard" >> build/$BUILD_FILE_NAME
  			    cat build/$BUILD_FILE_NAME
	            '''
            }
        }
        stage('Testing') { 
            steps {
            echo "testing the new hardware"
            sh '''
            test -f build/$BUILD_FILE_NAME
            grep -nr "Mainboard" build/$BUILD_FILE_NAME
            '''
            }
        }
    }
  post {
	  success {
	  	archiveArtifacts artifacts: "build/**"
	  }
  }
}
```
- CI because we don't want to wait for the weekend when issues accumulated , if we have more changes its better to test and catch the bug
- In case you need extra tools on jenkins agent(like npm), there are two approaches:
	- install all tools needed on the jenkins agent (like node/npm), manually
	- use docker (but you must install the plugin first docker pipeline)
```groovy
pipeline {
    agent any

    stages {
        stage('w/o docker') {
            steps {
                sh 'echo "without docker"'
            }
        }
        stage('w/ docker') {
            agent {
                docker {
                    image 'node:22-alpine'
                }
            }
            steps {
                sh 'echo "with docker"'
				sh 'npm --install'
            }
        }
    }
}
```
- troubleshooting docker agent:
	- bring down the containers `docker compose down`
	- bring them up `docker compose -d`
 
- In case you need to have  a shared workspace between the different steps you need to specify: `reuseNode true` it's false by default
```groovy
pipeline {
    agent any

    stages {
        stage('w/o docker') {
            steps {
                sh 'echo "without docker"'
            }
        }
        stage('w/ docker') {
            agent {
                docker {
                    image 'node:22-alpine'
					reuseNode true
                }
            }
            steps {
                sh 'echo "with docker"'
				sh 'npm --install'
            }
        }
    }
}
```
- Jenkins pipeline can be defined in a file called Jenkinsfile
- To make jenkins use the Jenkinsfile you create a new pipeline and instead of writing the groovy script you select `Pipeline script from SCM` > input Git as SCM > Input repository URL > make sure the branch is correct > Make sure the Script Path is correct
- Now we need to make the Jenkins file from git to build the project
```groovy
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
        stage("Test"){
            agent {
                docker {
                    image 'node:22-alpine'
					reuseNode true
                }
            }
            steps {
                echo "Test Stage"
                sh '''
                test -f build/index.html
                npm test
                '''
            }
        }
}
}

```
- Architecture with docker, the controller will find an agent with docker installed, the controller will clone the repositroy and store them in the workspace, the agent will start a docker container and map the workspace directory with the docker container, if they are not mapped to the worspace the files will be deleted once the job is done

- Report from jenkins is called JUnit, you can generate it like this for example
```groovy
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
        stage("Test"){
            agent {
                docker {
                    image 'node:22-alpine'
					reuseNode true
                }
            }
            steps {
                echo "Test Stage"
                sh '''
                test -f build/index.html
                npm test
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

```

- this is a comment in groovy `//`
- this is multiline comment in groovy
```groovy
/*
something
*/
```
- running E2E test using playwrigh
```groovy
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
                    image 'mcr.microsoft.com/playwright:v1.64.1-noble'
					reuseNode true
                    // args '-u root:root' // run as root, horrible solution, never do this
                }
            }
            steps {
                echo "Test Stage"
                sh '''
                npm install -g serve
                node_modules/.bin/serve -s build
				sleep 10
                npx playwright test
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
```