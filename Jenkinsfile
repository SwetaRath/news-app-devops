pipeline {
    agent { label 'slave2' }
    environment {
        // REPLACE these two values
        ART_URL = 'https://trial80p3wb.jfrog.io'   // <- replace with your Artifactory URL
        TARGET_REPO = '16-libs-release-local'                  // <- replace with repo name (generic-local, libs-release-local, etc.)

        // Jenkins credential containing JFrog API Key / token (Secret Text)
        ART_API_KEY = credentials('jfrog-token')
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'feature-2', url: 'https://github.com/SwetaRath/news-app-devops.git'
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests=false'
            }
        }
        stage('Run Tests') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Check User') {
    steps {
        sh 'whoami'
    }
}

        stage('Deploy WAR to Tomcat') {
            steps {
                sh '''
                    TOMCAT_PATH="/opt/tomcat10/webapps"
                    WAR_FILE="target/news-app.war"

                    echo "Cleaning old deployment..."
                    sudo rm -rf $TOMCAT_PATH/news-app $TOMCAT_PATH/news-app.war

                    echo "Copying new WAR..."
                    sudo cp $WAR_FILE $TOMCAT_PATH/

                    echo "Restarting Tomcat..."
                    pkill -f 'org.apache.catalina.startup.Bootstrap' || true
                    nohup $TOMCAT_PATH/../bin/startup.sh &
                '''
            }
        }
    }
    stage('Push the artifacts into JFrog Artifactory') {
            steps {
                script {
                    // Get the current date and time in the format: yyyy-MM-dd_HH-mm
                    def currentDate = new java.text.SimpleDateFormat("yyyy-MM-dd_HH-mm").format(new Date())

                    // Define the target path with the timestamp
                    def targetPath = "16-libs-release-local/${currentDate}/"

                    // Upload the built WAR to JFrog Artifactory with the timestamped path
                    rtUpload(
                        serverId: "jfrog",
                        spec: """{
                            "files": [
                                {
                                    "pattern": "${WAR_FILE}",
                                    "target": "${targetPath}"
                                }
                            ]
                        }"""
                    )
                }
            }
        }
    } // end stages

    post {
        success {
            echo 'Build and deployment completed successfully!'
        }
        failure {
            echo 'Build or deployment failed. Check logs for details.'
        }
    }
}
