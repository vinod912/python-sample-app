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
                    // Utilizing the cache-dir we setup with the PVC earlier
                    sh 'trivy fs --cache-dir /root/.cache/trivy --exit-code 0 --severity HIGH,CRITICAL .'
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
                        # python app.py  # Note: ensure app.py doesn't block the build
                    '''
                }
            }
        }

        stage('Static Code Quality') {
            steps {
                script {
                    // Use the container defined in your Cloud Pod Template
                    container('sonar-scanner') {
                        withSonarQubeEnv('sonarqube') {
                            echo "Forwarding static code analysis metrics to SonarQube..."
                            // No 'docker run' needed! We run natively in the scanner container.
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
