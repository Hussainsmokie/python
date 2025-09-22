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
                    sh '''
                        gitleaks detect --source=./projects --report-format=table --report-path=gitleaks-projects.txt --exit-code 1
                        gitleaks detect --source=./docs --report-format=table --report-path=gitleaks-docs.txt --exit-code 1
                    '''
                }
                sh 'ls -l gitleaks-*.txt || echo "No gitleaks report found"'
                archiveArtifacts artifacts: 'gitleaks-*.txt', onlyIfSuccessful: false
            }
        }

        stage('Trivy FS Scan') {
            steps {
                sh 'trivy fs --format table -o fs-report.html .'
                sh 'ls -l fs-report.html || echo "No Trivy report found"'
                archiveArtifacts artifacts: 'fs-report.html', onlyIfSuccessful: false
            }
        }

        stage('Download SonarQube Report') {
            steps {
                sh '''
                    curl -u $SONAR_TOKEN: "$SONAR_HOST_URL/api/issues/search?componentKeys=python-mini-projects&pageSize=500" -o sonar-issues.json
                '''
                sh 'ls -l sonar-issues.json || echo "No SonarQube report found"'
                archiveArtifacts artifacts: 'sonar-issues.json', onlyIfSuccessful: false
            }
        }

        stage('Email SonarQube Report') {
            steps {
                emailext (
                    subject: "SonarQube Report for Python Mini Projects",
                    body: "Hi Team,\n\nPlease find attached the latest SonarQube issues report.\n\nRegards,\nJenkins",
                    to: 'nousath1609@gmail.com',
                    attachmentsPattern: 'sonar-issues.json'
                )
            }
        }
    }

    post {
        always {
            echo "Finished SonarQube scan for all Python Mini Projects."
        }
    }
}
