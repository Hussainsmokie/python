pipeline {
    agent any  // Runs on any available Jenkins node

    environment {
        // Jenkins credential ID storing your Sonar token
        SONAR_TOKEN = credentials('SONAR_TOKEN_ID')
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout your GitHub repo containing all projects
                git url: 'https://github.com/Hussainsmokie/python.git', branch: 'main'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                // Use the SonarQube server configured in Jenkins ("sonar" must match your config name)
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
                // Wait for SonarQube Quality Gate and fail pipeline if not passed
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }

    post {
        always {
            echo "Finished SonarQube scan for all Python Mini Projects."
        }
    }
}
