pipeline {
    agent {
        node {
            label 'devops-agent'
        }
    }
    
    environment {
        // Force Git to ignore the handshake packet size issues
        GIT_SSL_NO_VERIFY = 'true'
    }
    
    options {
        timeout(time: 30, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
        ansiColor('xterm')
    }
    
    stages {
        stage('Initialize Workspace') {
            steps {
                // Set these globally in the pod to assist the checkout
                sh '''
                    git config --global http.sslVerify false
                    git config --global http.postBuffer 1048576000
                '''
            }
        }

        stage('SCA Dependency Scan') {
            steps {
                container('trivy') {
                    echo "Initializing Vulnerability Scanner..."
                    // Trivy uses the PVC we successfully bound
                    sh 'trivy fs --cache-dir /home/jenkins/.cache --exit-code 0 --severity HIGH,CRITICAL .'
                }
            }
        }

        stage('Python Build') {
            steps {
                container('python') {
                    sh '''
                        python3 -m venv venv
                        . venv/bin/activate
                        pip install --upgrade pip --trusted-host pypi.org --trusted-host files.pythonhosted.org
                        if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
                    '''
                }
            }
        }

        stage('Static Code Quality') {
            steps {
                script {
                    container('sonar-scanner') {
                        withSonarQubeEnv('sonarqube') {
                            sh '''
                                sonar-scanner \
                                -Dsonar.projectKey=python-sample-app \
                                -Dsonar.sources=. \
                                -Dsonar.host.url=http://sonarqube:9000 \
                                -Dsonar.scm.disabled=true
                            '''
                        }
                    }
                }
            }
        }
    }
}
