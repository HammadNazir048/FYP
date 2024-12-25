pipeline {
    agent any

    environment {
        APACHE_WEB_DIR = '/var/www/html/'   // Apache's document root
        AZURE_SERVER_IP = '20.118.206.34'  // Apache server IP
        SSH_CREDENTIALS_ID = 'azure-ssh-key'  // Replace with your Jenkins SSH credentials ID
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
                    // Debug: Print the contents of the index.html before deploying
                    echo 'Contents of index.html:'
                    sh 'cat index.html'

                    // Debug: Check current working directory to ensure correct file is being copied
                    echo "Current working directory: ${pwd()}"

                    // Copy index.html to the Apache server
                    sshagent(credentials: [SSH_CREDENTIALS_ID]) {
                        sh """
                            # Verify the remote Apache directory and check if it's correct
                            ssh azureuser@$AZURE_SERVER_IP 'echo "Remote Apache directory: $APACHE_WEB_DIR"'

                            # Copy index.html to Apache server
                            scp $LOCAL_HTML_FILE azureuser@$AZURE_SERVER_IP:$APACHE_WEB_DIR

                            # List files in the Apache directory to confirm the file copy
                            ssh azureuser@$AZURE_SERVER_IP 'ls -l $APACHE_WEB_DIR'
                        """
                    }
                }
            }
        }

        stage('Set Permissions') {
            steps {
                script {
                    // Set correct permissions on the index.html file to ensure Apache can serve it
                    sshagent(credentials: [SSH_CREDENTIALS_ID]) {
                        sh """
                            # Set proper ownership and permissions for Apache to serve the file
                            ssh azureuser@$AZURE_SERVER_IP 'sudo chown www-data:www-data $APACHE_WEB_DIR/index.html'
                            ssh azureuser@$AZURE_SERVER_IP 'sudo chmod 644 $APACHE_WEB_DIR/index.html'
                        """
                    }
                }
            }
        }

        stage('Restart Apache') {
            steps {
                script {
                    // Restart Apache to load the new index.html
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

        stage('Clear Apache Cache') {
            steps {
                script {
                    // Clear Apache cache to ensure the latest file is served
                    sshagent(credentials: [SSH_CREDENTIALS_ID]) {
                        sh """
                            # Clear Apache cache if any exists
                            ssh azureuser@$AZURE_SERVER_IP 'sudo systemctl reload apache2'
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
