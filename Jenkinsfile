pipeline {
    agent any
    
    parameters {
        string(name: 'KUBE_API_EP', defaultValue: 'https://k8s-ep-nlb-9d62b2cc3d3cb90a.elb.ap-south-1.amazonaws.com', description: 'provide the image tag to be deployed')
        string(name: 'cert_name', defaultValue: 'deploy-qa.crt', description: 'Add/Select the K8s CERT filename of the cluster you want to connect') 
    }
    environment {
        DOCKER= credentials('docker-creds')
        CERT = "$WORKSPACE/certs/${cert_name}"
    }
    stages{
        stage('CLEAN WORKSPACE'){
            steps{
                    cleanWs()
                }
            }
        stage ('git checkout') {
            steps {
                git 'https://github.com/akhilvijay15/K8_test.git'
            }
        }
        stage('docker-build'){
            steps{
                sh '''
                docker login -u $DOCKER_USR -p $DOCKER_PSW
                docker build -t regapp2 .
                docker tag regapp2:latest akhilvijay15/regapp2:latest
                docker push akhilvijay15/regapp2:latest
                '''
            }
        }
        stage("k8s-deployment")
		{
			steps{
                withCredentials([string(credentialsId: 'K8-TOKEN', variable: 'KUBE_API_TOKEN')]) {
                sh '''
                sh $WORKSPACE/certs/set-k8s-context.sh $KUBE_API_EP $KUBE_API_TOKEN $CERT
                kubectl apply -f . --insecure-skip-tls-verify
                kubectl get all --insecure-skip-tls-verify
                '''
                }
            }
        }
    }
}  
