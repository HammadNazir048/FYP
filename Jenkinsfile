pipeline {
    agent any

    environment {
        APACHE_WEB_DIR = '/var/www/html/'  // Apache's document root on Azure server
        AZURE_SERVER_IP = 'http://20.118.206.34/'  // Replace with your Azure server's IP
        SSH_CREDENTIALS_ID = 'azure-ssh-key'  // The ID of the SSH credentials in Jenkins
        LOCAL_HTML_FILE = 'index.html'  // Path to your local HTML file
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    // Checkout the code from GitHub
                    checkout scm
                }
            }
        }

        stage('Deploy HTML to Apache') {
            steps {
                script {
                    // Debug: Print the contents of the HTML file before deploying
                    sh 'cat index.html'

                    // Copy the HTML file to the Apache server
                    sshagent(credentials: [SSH_CREDENTIALS_ID]) {
                        sh """
                            # Debug: Verify the remote Apache directory
                            ssh azureuser@$AZURE_SERVER_IP 'echo "Remote Apache directory: $APACHE_WEB_DIR"'

                            # Copy index.html to Apache directory
                            scp $LOCAL_HTML_FILE azureuser@$AZURE_SERVER_IP:$APACHE_WEB_DIR

                            # Debug: List files in the remote Apache directory to confirm the copy
                            ssh azureuser@$AZURE_SERVER_IP 'ls -l $APACHE_WEB_DIR'
                        """
                    }
                }
            }
        }

        stage('Restart Apache') {
            steps {
                script {
                    // Restart Apache service to load the new index.html
                    sshagent(credentials: [SSH_CREDENTIALS_ID]) {
                        sh """
                            # Restart Apache to reflect changes
                            ssh azureuser@$AZURE_SERVER_IP 'sudo systemctl restart apache2'

                            # Debug: Check if Apache is running
                            ssh azureuser@$AZURE_SERVER_IP 'sudo systemctl status apache2'
                        """
                    }
                }
            }
        }

        stage('Notify') {
            steps {
                echo 'Deployment complete. Apache has been updated with the new index.html.'
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
