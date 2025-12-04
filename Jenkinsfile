pipeline {
    agent { label 'slave2' }

    environment {
        TOMCAT_PATH = "/opt/tomcat10/webapps"
        WAR_FILE = "target/news-app.war"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'feature-2', url: 'https://github.com/SwetaRath/news-app-devops.git'
            }
        }

        stage('Build') {
            steps {
                // This runs maven package (will run tests because -DskipTests=false)
                sh 'mvn clean package -DskipTests=false'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Deploy WAR to Tomcat') {
            steps {
                sh(script: '''
                    echo "Using TOMCAT_PATH=${TOMCAT_PATH}"
                    echo "WAR_FILE=${WAR_FILE}"

                    if [ ! -f "${WAR_FILE}" ]; then
                      echo "ERROR: WAR file ${WAR_FILE} not found"
                      exit 1
                    fi

                    echo "Cleaning old deployment..."
                    sudo rm -rf "${TOMCAT_PATH}/news-app" "${TOMCAT_PATH}/news-app.war" || true

                    echo "Copying new WAR..."
                    sudo cp "${WAR_FILE}" "${TOMCAT_PATH}/"

                    echo "Restarting Tomcat..."
                    # kill existing Tomcat process if running
                    pkill -f 'org.apache.catalina.startup.Bootstrap' || true

                    # start Tomcat (assumes ../bin/startup.sh is the startup script)
                    nohup "${TOMCAT_PATH}/../bin/startup.sh" > /dev/null 2>&1 &
                ''')
            }
        }

        stage('Push the artifacts into JFrog Artifactory') {
            steps {
                script {
                    // Get the current date and time in the format: yyyy-MM-dd_HH-mm
                    def currentDate = new java.text.SimpleDateFormat("yyyy-MM-dd_HH-mm").format(new Date())

                    // Define the target path with the timestamp
                    def targetPath = "16-libs-release/${currentDate}/"

                    // Upload the built WAR to JFrog Artifactory with the timestamped path
                    rtUpload(
                        serverId: "trial-artifactory",
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
