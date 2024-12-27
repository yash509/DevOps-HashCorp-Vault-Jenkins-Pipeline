# DevOps-HashCorp-Vault-Jenkins-Pipeline
The repository contains the HashiCorp Vault Installation commands and the Jenkins Pipeline


# For future references we can use the below links or notes or articles
https://blog.devops.dev/step-by-step-guide-using-hashicorp-vault-to-secure-aws-credentials-in-jenkins-ci-cd-pipelines-with-6a3971c63580
https://blog.devops.dev/end-to-end-aws-eks-provisioning-with-terraform-jenkins-ci-cd-and-hashicorp-vault-6caf11cb2d2c


# HashiCorp Installation Ubuntu Commands with Steps

sudo apt install -y apt-utils

wget -O - https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install vault
vault --version
vault



# Temporarily disable TLS to check if Vault starts correctly. Modify the listener in vault.hcl
sudo vi /etc/vault.d/vault.hcl

#add this as script
//
listener "tcp" {
  address     = "0.0.0.0:8200"
  tls_disable = 1 //this portion only
}
//

sudo systemctl enable vault
sudo systemctl start vault
sudo systemctl status vault



sudo ss -tuln | grep 8200

export VAULT_ADDR="http://<ec2-ip>:8200"
vault status

# Open a terminal and create the Vault service file using a text editor like vi or nano.
sudo vi /etc/systemd/system/vault.service

# Paste the following content into the file
//
[Unit]
Description=HashiCorp Vault - A tool for managing secrets
Documentation=https://developer.hashicorp.com/vault/docs
Requires=network-online.target
After=network-online.target

[Service]
User=root
Group=root
ExecStart=/usr/bin/vault server -config=/etc/vault.d/vault.hcl
ExecReload=/bin/kill --signal HUP $MAINPID
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
//

sudo systemctl daemon-reload

sudo systemctl enable vault
sudo systemctl start vault
sudo systemctl status vault




echo 'export VAULT_ADDR="http://<vault-server-ip>:8200"' >> ~/.bashrc 
source ~/.bashrc


# This below command will output:
	# 5 unseal keys (used to unseal Vault).	
	# Root token (used to log in and manage Vault)
//	
vault operator init
//



Important:
Save these keys securely (e.g., password manager, encrypted storage). Losing them can lock you out of Vault.
You’ll need at least 3 unseal keys to unseal Vault.

# Use the vault operator unseal command with three of the unseal keys generated during initialization:
vault operator unseal <UNSEAL_KEY_1>
vault operator unseal <UNSEAL_KEY_2>
vault operator unseal <UNSEAL_KEY_3>


# For login into the vault use the below command
vault login <root_token>




# Now, storing the AWS Credentials in Vault follow the below steps
1. Use the below command to Enable the KV Secrets Engine in Vault:
//
vault secrets enable -path=aws kv
//

2. Use the below command to Verify that the secrets engine is enabled:
//
vault secrets list
//

3. Use the below command to Add your AWS credentials (Access Key ID and Secret Access Key) to Vault:
//
vault kv put aws/terraform-project aws_access_key_id=<your-access-key-id> aws_secret_access_key=<your-secret-access-key>
//
//aws: Represents a namespace or folder for AWS-related secrets in Vault.
//terraform-project: Represents a specific Terraform project using AWS credentials.

4. Use the below command to Verify that the secret is stored:
//
vault kv get aws/terraform-project
//

5. Use the below commands to Configure Vault Authentication:
5.1. Use the below command to Enable AppRole Authentication:
//
vault auth enable approle
//

5.2. Create a Policy for Access: Create a policy file, e.g., aws-policy.hcl:
//
path "aws/terraform-project" {
    capabilities = ["read"]
}
//
Apply the policy by using the below command:
//
vault policy write aws-policy aws-policy.hcl
vault write auth/approle/role/cicd-role token_policies="aws-policy"
//

5.3. Create an AppRole using the below command:
//
vault write auth/approle/role/cicd-role
	token_policies="aws-policy"
	secret_id_ttl=24h \
    token_ttl=1h \
    token_max_ttl=4h
//
	
5.4. Retrieve Role ID and Secret ID using the below command:
//
vault read auth/approle/role/cicd-role/role-id
vault write -f auth/approle/role/cicd-role/secret-id
//
