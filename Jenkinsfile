pipeline {
    agent { label 'slave2' }

    environment {
        ART_URL = 'https://trial80p3wb.jfrog.io'
        TARGET_REPO = '16-libs-release-local'
        ART_API_KEY = credentials('jfrog-token')
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

        rtUpload()
rtDownload()
rtServer()
rtBuildInfo()
// end stages

    post {
        success {
            echo 'Build and deployment completed successfully!'
        }
        failure {
            echo 'Build or deployment failed. Check logs for details.'
        }
    }
}
}
