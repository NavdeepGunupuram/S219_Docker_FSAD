pipeline {
    agent any

    environment {
        APP_NAME = "ecommerce"
        TOMCAT_URL = "http://localhost:9090"      // change if Tomcat elsewhere
        TOMCAT_MANAGER = "${TOMCAT_URL}/manager/text"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/NavdeepGunupuram/S219_Docker_FSAD.git'
            }
        }

        stage('Build Backend') {
            steps {
                dir('backend') {
                    sh 'mvn clean package -DskipTests'
                }
            }
        }

        stage('Build Frontend') {
            steps {
                dir('frontend') {
                    sh 'npm ci'
                    sh 'npm run build'
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                sh 'docker-compose build'
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'tomcat-cred', usernameVariable: 'TOMCAT_USER', passwordVariable: 'TOMCAT_PASS')]) {
                    sh '''
                        WAR=$(ls backend/target/*.war | head -n1)
                        if [ -z "$WAR" ]; then
                          echo "No WAR found in backend/target"
                          exit 1
                        fi
                        echo "Deploying $WAR to Tomcat..."
                        curl --fail --show-error --user $TOMCAT_USER:$TOMCAT_PASS \
                          -T "$WAR" "${TOMCAT_MANAGER}/deploy?path=/${APP_NAME}&update=true"
                    '''
                }
            }
        }

        stage('Start Services') {
            steps {
                sh 'docker-compose up -d'
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful! Frontend & Backend are running."
        }
        failure {
            echo "❌ Deployment failed. Check console output."
        }
    }
}
