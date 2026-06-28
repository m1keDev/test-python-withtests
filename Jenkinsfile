pipeline {
    agent any

    environment {
        SONAR_PROJECT_KEY = 'test-python-withtests'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo "Building commit: ${env.GIT_COMMIT}"
            }
        }

        stage('Install') {
            steps {
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install --upgrade pip
                    pip install pytest pytest-cov 2>/dev/null || true
                    if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    . venv/bin/activate
                    set +e
                    python3 -m pytest --cov=app --cov-report=xml:coverage.xml --junitxml=test-results.xml -v
                    EXIT=$?
                    set -e
                    if [ "$EXIT" = "5" ]; then
                        echo "No tests found -- skipping test execution"
                    elif [ "$EXIT" != "0" ]; then
                        exit "$EXIT"
                    fi
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        sonar-scanner \
                            -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                            -Dsonar.sources=app \
                            -Dsonar.python.coverage.reportPaths=coverage.xml \
                            -Dsonar.exclusions=venv/**,node_modules/** \
                            -Dsonar.scm.revision=${GIT_COMMIT}
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 3, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: false
                }
            }
        }
    }

    post {
        always {
            script {
                if (fileExists('test-results.xml')) {
                    junit 'test-results.xml'
                }
            }
            echo "Pipeline finished for commit: ${env.GIT_COMMIT}"
        }
    }
}
