pipeline {
    agent any
    stages {
        stage('login into account') {
            steps {
                sh '''
                   az login -u 'rafael.martinez@globant.com' -p '880917La@'
                   az account set -s a265068d-a38b-40a9-8c88-fb7158ccda23
                   '''
            }
        }
        stage('creating the resource Group') {
            steps {
                sh 'az group create -n sqlTerraform-RG-MI -l eastus'
            }
        }
        stage('creating the storage account') {
            steps {
                sh 'az storage account create -n sqlmitfstatestgtest -g sqlTerraform-RG-MI -l eastus'
            }
        }
        stage('creating a sqlmitfstate container') {
            steps {
                sh '''
                   az storage container create -n sqlmitfstate --account-name sqlmitfstatestgtest
                   '''
            }
        }
        stage('creating the KeyVault') {
            steps {
                sh '''
                   az keyvault create -n sqlmitfstatekv-test-01 -g sqlTerraform-RG-MI -l eastus
                   '''
            }
        }
        stage('Creating a SAS Token for the storage account, storing in KeyVault') {
            steps {
                sh '''
                   az storage container generate-sas --account-name sqlmitfstatestgtest --expiry 2021-04-04 --name sqlmitfstate --permissions dlrw --output json | xargs az keyvault secret set --vault-name sqlmitfstatekv-test-01 --name TerraformSASToken --value
                   '''
            }
        }
        stage('creating an ssh key') {
            steps {
                sh '''
                   /usr/bin/expect -c 'expect "Overwrite (y/n)?\r" { spawn ssh-keygen -f ~/.ssh/id_rsa_terraform -q -N ""; send "Y\r" }'
                   '''
            }
        }
        stage('creating a Service Principal ') {
            steps {
                sh '''
                   SP=$(az ad sp create-for-rbac -n "SqlMiTerraformSP")
                   az keyvault secret set --vault-name sqlmitfstatekv-test-01 --name LinuxSSHPubKey -f ~/.ssh/id_rsa_terraform.pub > /dev/null
                   az keyvault secret set --vault-name sqlmitfstatekv-test-01 --name spn-id --value $(echo $SP | jq -r '.appId') > /dev/null
                   az keyvault secret set --vault-name sqlmitfstatekv-test-01 --name spn-secret --value $(echo $SP | jq -r '.password') > /dev/null
                   az keyvault secret set --vault-name sqlmitfstatekv-test-01 --name spn-tenant --value $(echo $SP | jq -r '.tenant') > /dev/null
                   '''
            }
        }
    }
}
