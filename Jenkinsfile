pipeline {
    agent any
    environment {
        SONAR_RUNNER_HOME = '/home/jenkins/sonar-scanner'
        PATH = "${SONAR_RUNNER_HOME}/bin:${env.PATH}"
        ZAP_API_KEY = credentials('ZAP_API_KEY') // Use the credentials ID you set in Jenkins
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/rthoma38/scanner.git'
            }
         }
        stage('Fix Feature Names') {
            steps {
                sh 'python fix_feature_names.py'
            }
        }
        stage('Install Dependencies') {
            steps {
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install python-owasp-zap-v2.4 scikit-learn pandas numpy joblib
                '''
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube Scanner') {
                    sh 'sonar-scanner -Dsonar.projectKey=SonarQube_Analysis -Dsonar.sources=. -Dsonar.exclusions=venv/** -Dsonar.host.url=http://localhost:9000 -Dsonar.login=${SONARQUBE_TOKEN}'
                }
            }
        }
        stage('Dynamic Vulnerability Scan - OWASP ZAP') {
            steps {
                sh '''
                    . venv/bin/activate
                    python3 zap_scan.py
                '''
            }
            post {
                always {
                    archiveArtifacts artifacts: 'zap_report.html', allowEmptyArchive: true
                }
            }
        }    
        stage('Vulnerability Scan - Trivy') {
            steps {
                sh 'docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:latest image web-app'
            }
        }
        stage('Generate Dataset') {
            steps {
                sh '''
                    . venv/bin/activate
                    python generate_network_activity.py
                '''
            }
        }
        stage('Train Model') {
            steps {
                sh '''
                    . venv/bin/activate
                    python train_model.py
                '''
            }
        }
        stage('Real-time Detection') {
            steps {
                sh '''
                    . venv/bin/activate
                    python real_time_detection.py
                '''
            }
        }
    }
}
