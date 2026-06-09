pipeline {
    agent any

    tools {
        // Must match the name defined in Manage Jenkins -> Tools
        maven 'Maven_3' 
    }

    environment {
        // URL of your Tomcat Manager text interface
        TOMCAT_URL = 'http:localhost:8080/manager/text'
        TOMCAT_CONTEXT = '/demo-java-war' // The context path your app will run on
    }

    stages {
        stage('Checkout') {
            steps {
                // Clones the repository specified in your prompt
                git branch: 'main', url: 'https://github.com/akshaykadam-45/demo-java-war.git'
            }
        }

        stage('Build') {
            steps {
                // Compiles code, runs tests, and packages the application into a .war file
                sh 'mvn clean package'
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                // Securely fetches Tomcat credentials and deploys using curl to the Tomcat Manager API
                withCredentials([usernamePassword(credentialsId: 'tomcat-manager-creds', passwordVariable: 'TOMCAT_PASS', usernameVariable: 'TOMCAT_USER')]) {
                    script {
                        // Locate the generated war file (usually target/*.war)
                        def warFile = sh(script: 'ls target/*.war', returnStdout: true).trim()
                        
                        echo "Undeploying previous application if exists..."
                        // We ignore errors on undeploy because it will fail if the application isn't already deployed
                        sh "curl -u ${TOMCAT_USER}:${TOMCAT_PASS} \"${TOMCAT_URL}/undeploy?path=${TOMCAT_CONTEXT}\" || true"
                        
                        echo "Deploying ${warFile} to Tomcat..."
                        sh "curl -u ${TOMCAT_USER}:${TOMCAT_PASS} --upload-file ${warFile} \"${TOMCAT_URL}/deploy?path=${TOMCAT_CONTEXT}&update=true\""
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Application built and deployed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the logs above.'
        }
    }
}
