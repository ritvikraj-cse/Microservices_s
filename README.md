## Microservices Multibranch Pipeline CI/CD Steps

—————————————————

- Configure SG
- IAM user, Add policies, access keys (Refer to `infra-step` branch)
- EC2 - Ubuntu Server 20.0, t2.large, SG, 25GB

—————————————————

### SSH to the Ubuntu Server
1. Update the package list:
    ```bash
    sudo apt update
    ```

2. Install AWS CLI, KUBECTL, EKSCTL (Refer to `infra-step` branch):
    ```bash
    mkdir scripts
    cd scripts
    vi scr.sh
    # Paste installation script into scr.sh
    sudo chmod +x scr.sh
    ./scr.sh
    ```

3. Configure AWS CLI:
    ```bash
    aws configure
    ```

4. Create EKS Cluster through AWS CLI (Refer to `infra-step` branch)

5. Install Java 17:
    ```bash
    sudo apt install openjdk-17-jre-headless -y
    ```

6. Install Jenkins and configure (Access at `<ubuntu_public_ip>:8080`)

7. Install Docker:
    ```bash
    sudo apt install docker.io -y
    sudo chmod 666 /var/run/docker.sock
    ```

—————————————————

### On Jenkins - CI
1. Install Docker and Kubernetes plugins, also Multibranch Scan Webhook Trigger plugin.

2. Configure Docker installations:
    - Manage Jenkins -> Tools -> Configure Docker installations

3. Add credentials:
    - Manage Jenkins -> Credentials -> System -> Global -> Add Credentials
        - For Docker:
            - Kind: Username with password
            - Username: 
            - Password: 
            - ID: docker-red
        - For GitHub:
            - Kind: Username with password
            - Username: 
            - Password: 
            - ID: git-cred

4. Create a Multibranch Pipeline:
    - New item -> Multibranch pipeline
    - Branch Sources:
        - Git -> Provide project repo, git-cred
    - Build Configuration:
        - Mode: By Jenkinsfile
        - Script Path: Jenkinsfile
    - Scan Multibranch Pipeline Triggers:
        - Scan by webhook -> Apply

—————————————————

### GitHub
- Settings -> Webhooks -> Add Webhook
    - Payload URL:
    - Content type: JSON
    - Event: Just the push event

—————————————————

### On Ubuntu Server - K8s Configuration
1. Create namespace:
    ```bash
    kubectl create namespace webapps
    ```

2. Create Service Account (Refer to `infra-step` branch)

3. Create Role

4. Bind the Role to the Service Account

5. Generate token using the Service Account in the namespace:
    ```bash
    kubectl describe secret {secret_name} -n webapps
    ```
   - Copy the token

—————————————————

### On Jenkins - CD
1. Add credentials:
    - Manage Jenkins -> Credentials -> System -> Global -> Add Credentials
        - Kind: Secret text
        - Secret: {K8s token}
        - ID: K8-token

2. Create a dummy pipeline:
    - New item -> Dummy pipeline -> Generate pipeline script

3. Copy the script and save it in `Jenkinsfile` in the main branch of the microservice project -> Commit changes.

4. Delete the dummy pipeline.

Automatically, the main pipeline will be executed, where all 11 microservices get executed and deployed to EKS.
