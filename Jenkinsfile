pipeline {
    agent { label 'slave2' }

    environment {
        TOMCAT_PATH = "/opt/tomcat10/webapps"
        WAR_FILE    = "target/news-app.war"
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
                    // Ensure WAR exists on the agent before attempting upload
                    if (!fileExists(env.WAR_FILE)) {
                        error "Artifact not found: ${env.WAR_FILE}"
                    }

                    // Get the current date and time in the format: yyyy-MM-dd_HH-mm
                    def currentDate = new java.text.SimpleDateFormat("yyyy-MM-dd_HH-mm").format(new Date())

                    // Define the target path with the timestamp
                    def targetPath = "16-libs-release/${currentDate}/"

                    // Resolve the Artifactory server (ensure 'ART' exists as a configured server ID)
                    // Replace 'ART' with your actual server ID if different
                    def server = Artifactory.server('ART')

                    // local war variable (use env to get pipeline env var)
                    def war = env.WAR_FILE

                    // Define upload spec (rtUpload expects a spec string or map)
                    def uploadSpec = """{
                        "files": [
                            {
                                "pattern": "${war}",
                                "target": "${targetPath}"
                            }
                        ]
                    }"""

                    // Upload to Artifactory
                    rtUpload server: server, spec: uploadSpec
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
