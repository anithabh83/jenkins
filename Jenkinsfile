pipeline {
    agent { label 'new' }

    environment {
        FOLDER = "javaapp-standalone"
        SONAR_TOKEN = credentials('sonarqube-cred')
    }

    stages {

        stage('Checkout') {
            steps {
                git url: 'https://github.com/anithabh83/jenkins.git'
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

        stage('Build') {
            when { branch 'master' } // Only build on master
            steps {
                sh """
                cd ${FOLDER}
                mvn clean package -DskipTests
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

        stage('Approval Before Deployment') {
            when { branch 'master' } // Only on master
            steps {
                script {
                    timeout(time: 5, unit: 'MINUTES') {
                        input message: "Approve Deployment?", ok: "Deploy Now"
                    }
                }
            }
        }

        stage('Deploy') {
            when { branch 'master' } // Only on master
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

