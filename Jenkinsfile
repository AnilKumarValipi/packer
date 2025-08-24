pipeline {


stage('Checkout') {
steps {
// If using Multibranch or Pipeline from SCM, Jenkins will auto-checkout.
// For freestyle Pipeline, uncomment the next line and set your repo URL.
// git branch: 'main', url: 'https://github.com/your-org/ami-pipeline.git'
echo 'SCM checked out by Jenkins.'
}
}


stage('Packer Init') {
steps {
// If your packer files are in ./packer use this:
sh 'packer init packer'
// If your packer files are in repo root, use: sh 'packer init .'
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
// Build AMI and capture AMI ID from machine-readable output
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