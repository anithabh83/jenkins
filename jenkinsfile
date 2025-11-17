pipeline {
    agent { label 'new' }

    environment {
        FOLDER = "javaapp-standalone"
        SONAR_TOKEN = credentials('sonarqube-cred')
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Test') {
            steps {
                sh """
                cd ${FOLDER}
                mvn clean test
                """
            }
        }

        stage('SonarQube Analysis') {
            steps {
                dir("${FOLDER}") {
                    withSonarQubeEnv('sonar') {
                        sh """
                            mvn clean verify sonar:sonar \
                                -Dsonar.projectKey=java-app \
                                -Dsonar.projectName=java-app
                        """
                    }
                }
            }
        }

        /* -------------------------------
           MAIN BRANCH ONLY: BUILD + DEPLOY 
        ---------------------------------- */
        stage('Build') {
            when { branch 'master' }
            steps {
                sh """
                cd ${FOLDER}
                mvn clean package -DskipTests
                """
            }
        }

        stage('Approval Before Deployment') {
            when { branch 'master' }
            steps {
                script {
                    timeout(time: 5, unit: 'MINUTES') {
                        input message: "Approve Deployment?", ok: "Deploy Now"
                    }
                }
            }
        }

        stage('Deploy') {
            when { branch 'master' }
            steps {
                sh """
                cd ${FOLDER}/target

                if pgrep -f "java -jar java-sample-21-1.0.0.jar" > /dev/null; then
                    pkill -f "java -jar java-sample-21-1.0.0.jar"
                    echo "App was running and has been killed"
                else
                    echo "App is not running"
                fi

                JENKINS_NODE_COOKIE=dontKillMe nohup java -jar java-sample-21-1.0.0.jar > app.log 2>&1 &
                echo "New app started"
                """
            }
        }
    }

    post {
        always {
            echo "Cleaning workspace"
            cleanWs()
        }
    }
}
