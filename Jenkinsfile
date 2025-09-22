pipeline {
    agent any

    environment {
        SONAR_TOKEN = credentials('SONAR_TOKEN_ID')
    }

    stages {
        stage('Git Checkout') {
            steps {
                git url: 'https://github.com/Hussainsmokie/python.git', branch: 'main'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''
                        /opt/sonar-scanner/bin/sonar-scanner \
                          -Dsonar.projectKey=python-mini-projects \
                          -Dsonar.projectName="Python Mini Projects" \
                          -Dsonar.sources=projects \
                          -Dsonar.host.url=$SONAR_HOST_URL \
                          -Dsonar.login=$SONAR_TOKEN \
                          -Dsonar.sourceEncoding=UTF-8
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Git Leaks') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    sh 'gitleaks detect --source=./projects --exit-code 1'
                    sh 'gitleaks detect --source=./docs --exit-code 1'
                }
            }
        }

        stage('Trivy FS Scan') {
            steps {
                sh 'trivy fs --format table -o fs-report.html .'
            }
        }
    }

    post {
        always {
            echo "Finished SonarQube scan for all Python Mini Projects."
        }
    }
}
