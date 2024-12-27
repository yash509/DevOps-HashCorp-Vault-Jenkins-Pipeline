pipeline {
    agent any

    environment {
        VAULT_URL = 'http://:8200/' // Vault server URL needs to be changed everytime manually
    }

    stages {
        
        stage("Debug Vault Credentials") {
            steps {
                script {
                    echo "Verifying Vault Credentials Configuration..."
                    withCredentials([
                        string(credentialsId: 'VAULT_URL', variable: 'VAULT_URL'),
                        string(credentialsId: 'vault-role-id', variable: 'VAULT_ROLE_ID'),
                        string(credentialsId: 'vault-secret-id', variable: 'VAULT_SECRET_ID')
                    ]) {
                        echo "Vault Role ID is available"
                        echo "Vault Secret ID is available"
                    }
                }
            }
        }

        stage("Test Vault Connectivity and Login") {
            steps {
                script {
                    withCredentials([
                        string(credentialsId: 'VAULT_URL', variable: 'VAULT_URL'),
                        string(credentialsId: 'vault-role-id', variable: 'VAULT_ROLE_ID'),
                        string(credentialsId: 'vault-secret-id', variable: 'VAULT_SECRET_ID')
                    ]) {
                        echo "Testing Vault Connectivity..."
                        sh '''
                        # Set Vault address
                        export VAULT_ADDR="${VAULT_URL}"

                        # Log into Vault using AppRole
                        echo "Logging into Vault using AppRole..."
                        VAULT_TOKEN=$(vault write -field=token auth/approle/login role_id=${VAULT_ROLE_ID} secret_id=${VAULT_SECRET_ID})
                        echo "Vault Login Successful"

                        # Verify connectivity
                        vault status
                        '''
                    }
                }
            }
        }
        
        stage("Fetch Credentials from Vault") {
            steps {
                script {
                    withCredentials([
                        string(credentialsId: 'VAULT_URL', variable: 'VAULT_URL'),
                        string(credentialsId: 'vault-role-id', variable: 'VAULT_ROLE_ID'),
                        string(credentialsId: 'vault-secret-id', variable: 'VAULT_SECRET_ID')
                        ]) 
                        {
                            echo "Fetching GitHub and AWS credentials from Vault..."
                            // Fetch secrets with error handling
                            sh '''
                            # Set Vault server URL
                            export VAULT_ADDR="${VAULT_URL}"
                            
                            # Log into Vault using AppRole
                            echo "Logging into Vault..."
                            VAULT_TOKEN=$(vault write -field=token auth/approle/login role_id=${VAULT_ROLE_ID} secret_id=${VAULT_SECRET_ID} || { echo "Vault login failed"; exit 1; })
                            export VAULT_TOKEN=$VAULT_TOKEN
                            
                            # Fetch GitHub token
                            # echo "Fetching GitHub Token..."
                            # GIT_TOKEN=$(vault kv get -field=pat secret/github || { echo "Failed to fetch GitHub token"; exit 1; })
                            
                            # Fetch AWS credentials
                            echo "Fetching AWS Credentials..."
                            AWS_ACCESS_KEY_ID=$(vault kv get -field=aws_access_key_id aws/terraform-project || { echo "Failed to fetch AWS Access Key ID"; exit 1; })
                            AWS_SECRET_ACCESS_KEY=$(vault kv get -field=aws_secret_access_key aws/terraform-project || { echo "Failed to fetch AWS Secret Access Key"; exit 1; })
                            
                            # Export credentials to environment variables
                            echo "Exporting credentials to environment..."
                            echo "export GIT_TOKEN=${GIT_TOKEN}" >> vault_env.sh
                            echo "export AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}" >> vault_env.sh
                            echo "export AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}" >> vault_env.sh
                            '''
                            // Load credentials into environment
                            sh '''
                            echo "Loading credentials into environment..."
                            . ${WORKSPACE}/vault_env.sh
                            echo "Credentials loaded successfully."
                            '''
                        }
                }
            }
        }
        
        stage('Checkout from Git') {                        
            steps {                                       
                git branch: 'main', url: 'https://github.com/yash509/DevSecOps-ChrmeLog-Wp-Deployment.git'
            }
        }
    }
}
