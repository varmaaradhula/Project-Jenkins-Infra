pipeline{
    agent any
    environment {
        GIT_REPO_URL = 'https://github.com/varmaaradhula/Project-Jenkins-Infra.git'
        GIT_BRANCH = 'master'
        S3_BUCKET = 'vprofile-terraform-state'
        TF_PATH = 'terraform'
        S3_KEY = 'terraform/state/terraform.tfstate'
        AWS_REGION = 'eu-west-2'
       // AWS_CREDENTIALS = credentials('awscreds')
        TERRAFORM_APPLY_STATUS = 'false'
        GET_KUBECONFIG_STATUS = 'false'
    }

    stages{

        stage('Checkout code from Git Repo'){

            steps{
                echo "Checking out terraform repo from GitHub"

                git branch: "${GIT_BRANCH}", url: "${GIT_REPO_URL}"
            }
        }

        stage('Config AWS credentials'){
            steps {
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding', 
                     credentialsId: 'awscreds'] // Replace with your Jenkins credential ID
                ]) {
                    script {
                        // AWS credentials will be available as environment variables
                        sh '''
                        echo "Configuring AWS CLI"
                        aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                        aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                        aws configure set default.region ${AWS_REGION}
                        '''
                    }
                }
            }
        }

        stage('Intiate Terraform'){
            steps {
                echo 'Intiating Terraform'
                dir("${TF_PATH}")
                        sh '''
                        terraform init \
                          -backend-config="bucket=${S3_BUCKET}" \
                          -backend-config="key=terraform/state.tfstate" \
                          -backend-config="region=$AWS_REGION"
                          '''
            }
        }
        stage('Terraform Validate') {
            steps {
                echo 'Validating Terraform configuration...'
                dir("${TF_PATH}")
                sh 'terraform validate'
            }
        }

        stage('Terraform Plan') {
            steps {
                echo 'Planning Terraform and getting output file'
                dir("${TF_PATH}")
                        sh '''
                        terraform plan \
                          -var="aws_region=$AWS_REGION" \
                          -out=tfplan.out
                        '''
                    }
                }
        }
}