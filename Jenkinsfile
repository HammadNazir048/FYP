pipeline {
    agent any

    environment {
        // Define the directory where Apache serves the files on the Azure server
        APACHE_WEB_DIR = '/var/www/html/'  // Update if your Apache config differs
        AZURE_SERVER_IP = '20.118.206.34'
        SSH_CREDENTIALS_ID = 'azure-ssh-key'  // The ID of your stored SSH key in Jenkins
        LOCAL_HTML_FILE = 'index.html'  // Path to your local HTML file
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    // Checkout the code from GitHub (or your version control system)
                    checkout scm
                }
            }
        }

        stage('Deploy HTML to Apache') {
            steps {
                script {
                    // Use SSH credentials to copy the HTML file to the Apache server on Azure
                    sshagent(credentials: [SSH_CREDENTIALS_ID]) {
                        sh """
                            scp $LOCAL_HTML_FILE azureuser@$AZURE_SERVER_IP:$APACHE_WEB_DIR
                        """
                    }
                }
            }
        }

        stage('Restart Apache') {
            steps {
                script {
                    // Restart Apache to reflect the changes on the Azure server
                    sshagent(credentials: [SSH_CREDENTIALS_ID]) {
                        sh """
                            ssh azureuser@$AZURE_SERVER_IP 'sudo systemctl restart apache2'
                        """
                    }
                }
            }
        }

        stage('Notify') {
            steps {
                echo 'Deployment complete. The Apache server is updated with the new index.html.'
            }
        }
    }

    post {
        success {
            echo 'Job succeeded!'
        }
        failure {
            echo 'Job failed.'
        }
    }
}
