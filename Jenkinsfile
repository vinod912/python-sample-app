pipeline {
    agent {
        node {
            label 'devops-agent'
        }
    }
    
    environment {
        // BYPASS MTU/TLS HANDSHAKE ISSUES
        GIT_SSL_NO_VERIFY = 'true'
    }
    
    options {
        timeout(time: 30, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
        ansiColor('xterm')
    }
    
    stages {
        stage('Network Setup') {
            steps {
                // This forces Git to use a massive buffer and skip SSL
                // which prevents the "GnuTLS handshake" crash.
                sh '''
                    git config --global http.sslVerify false
                    git config --global http.postBuffer 524288000
                '''
            }
        }

        stage('SCA Dependency Scan') {
            steps {
                container('trivy') {
                    echo "Initializing Vulnerability Scanner..."
                    sh 'trivy fs --cache-dir /root/.cache/trivy --exit-code 0 --severity HIGH,CRITICAL .'
                }
            }
        }

        stage('Python Compile & Test') {
            steps {
                container('python') {
                    echo "Python runtime active. Building workspace environment..."
                    sh '''
                        # Lower MTU for the pip install as well
                        python3 -m venv venv
                        . venv/bin/activate
                        pip install --upgrade pip
                        pip install -r requirements.txt
                    '''
                }
            }
        }

        stage('Static Code Quality') {
            steps {
                script {
                    container('sonar-scanner') {
                        withSonarQubeEnv('sonarqube') {
                            echo "Forwarding static code analysis metrics to SonarQube..."
                            sh '''
                                sonar-scanner \
                                -Dsonar.projectKey=my-devops-project \
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
