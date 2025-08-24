pipeline {
    agent any

    environment {
        AWS_REGION   = "us-east-1"                 // Region
        SUBNET_ID    = "subnet-0d5d4f3a59fb04830"  // your subnet ID
        SSH_SG_ID    = "sg-0df3fcd13460aa112"      // your security group ID
        PACKER_FILE  = "packer/nginx.json"         // change to tomcat.json if needed
    }

    stages {

        stage('Checkout') {
            steps {
                // If using "Pipeline from SCM", Jenkins auto-checks out your repo.
                echo 'SCM checked out by Jenkins.'
            }
        }

        stage('Packer Init') {
            steps {
                // If packer files are in ./packer dir
                sh 'packer init packer'
                // If files are at repo root, use: sh 'packer init .'
            }
        }

        stage('Packer Validate') {
            steps {
                sh '''
                packer validate \
                -var aws_region=$AWS_REGION \
                -var subnet_id=$SUBNET_ID \
                -var ssh_sg_id=$SSH_SG_ID \
                $PACKER_FILE
                '''
            }
        }

        stage('Bake AMI') {
            steps {
                script {
                    // Build AMI and capture AMI ID
                    def output = sh(
                        script: '''
                        set -e
                        packer build -machine-readable \
                        -var aws_region=$AWS_REGION \
                        -var subnet_id=$SUBNET_ID \
                        -var ssh_sg_id=$SSH_SG_ID \
                        $PACKER_FILE \
                        | tee packer_build.log
                        ''',
                        returnStdout: true
                    )

                    def amiLine = output.readLines().find { it.contains('artifact,0,id') }
                    if (!amiLine) {
                        error 'Could not parse AMI ID from Packer output. See packer_build.log.'
                    }

                    def amiId = amiLine.split(",").last().split(":").last().trim()
                    echo "✅ New AMI created: ${amiId}"
                }
            }
        }
    }
}
