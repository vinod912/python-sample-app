pipeline {
    agent {
        node {
            label 'devops-agent'
        }
    }
    
    options {
        timeout(time: 30, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
        ansiColor('xterm')
    }
    
    stages {
        stage('SCA Dependency Scan') {
            steps {
                container('trivy') {
                    echo "Initializing Vulnerability Scanner..."
                    sh 'trivy fs --exit-code 0 --severity HIGH,CRITICAL .'
                }
            }
        }

        stage('Python Compile & Test') {
            steps {
                container('python') {
                    echo "Python runtime active. Building workspace environment..."
                    sh '''
                        python3 -m venv venv
                        . venv/bin/activate
                        pip install --upgrade pip
                        pip install -r requirements.txt
                        python app.py
                    '''
                }
            }
        }

        stage('Static Code Quality') {
            steps {
                script {
                    withSonarQubeEnv('sonarqube') {
                        echo "Forwarding static code analysis metrics to SonarQube..."
                        sh '''
                            docker run --rm --user 0:0 --network devops-network \
                            -v ${WORKSPACE}:/usr/src \
                            sonarsource/sonar-scanner-cli \
                            -Dsonar.projectKey=my-devops-project \
                            -Dsonar.sources=. \
                            -Dsonar.host.url=http://sonarqube:9000 \
                            -Dsonar.working.directory=/usr/src/.scannerwork \
                            -Dsonar.projectBaseDir=/usr/src
                        '''
                    }
                }
            }
        }
    }
}
