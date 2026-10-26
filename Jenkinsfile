pipeline {
    agent any

    environment {
        DEPLOY_DIR = '/opt/learnsphere-deploy'
        INVENTORY = '/opt/learnsphere-deploy/ansible/inventory.ini'
        PLAYBOOK = '/opt/learnsphere-deploy/ansible/deploy-app.yml'
    }

    stages {
        stage('Checkout Source') {
            steps {
                checkout scm
            }
        }

        stage('Validate Source') {
            steps {
                sh '''
                    set -eu
                    test -f app/index.html
                    test -f app/Dockerfile
                    echo "LearnSphere source validated."
                '''
            }
        }

        stage('Build Container Image') {
            steps {
                sh '''
                    set -eu
                    podman build \
                        -t learnsphere:${BUILD_NUMBER} \
                        -t learnsphere:latest \
                        app
                '''
            }
        }

        stage('Export Container Image') {
            steps {
                sh '''
                    set -eu
                    podman save \
                        -o "$DEPLOY_DIR/app/learnsphere.tar" \
                        localhost/learnsphere:latest
                    test -s "$DEPLOY_DIR/app/learnsphere.tar"
                    ls -lh "$DEPLOY_DIR/app/learnsphere.tar"
                '''
            }
        }

        stage('Deploy to App Servers') {
            steps {
                sh '''
                    set -eu
                    ansible-playbook \
                        -i "$INVENTORY" \
                        "$PLAYBOOK"
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    set -eu
                    for url in http://3.24.136.84 http://3.107.68.232; do
                        echo "Checking $url"
                        curl --fail --silent --show-error \
                            --output /dev/null \
                            --write-out "HTTP %{http_code}\\n" \
                            "$url"
                    done
                '''
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: LearnSphere deployed to both application servers.'
        }
        failure {
            echo 'FAILURE: Check Console Output for the failed stage.'
        }
        always {
            echo 'Pipeline execution completed.'
        }
    }
}
