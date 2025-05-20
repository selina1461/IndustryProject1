pipeline {
    agent none
    environment{
	DOCKER_HUB_CREDS = credentails('docker-credentials')
	KUBECONFIG = credentials('kubeconfig-credentials')
	IMAGE_NAME = 'selinamjo1/jenkins/inbound-agent'
        IMAGE_TAG = "latest"

    }
    stages {
        stage('Compile') {
            agent {
                docker {
                    image 'maven:3.8.6-openjdk-11'
                    label 'docker-agent'
                }
            }
            steps {
                sh 'mvn clean compile'
            }
        }
        
        stage('Test') {
            agent {
                docker {
                    image 'maven:3.8.6-openjdk-11'
                    label 'docker-agent'
                }
            }
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }
        
        stage('Package') {
            agent {
                docker {
                    image 'maven:3.8.6-openjdk-11'
                    label 'docker-agent'
                }
            }
            steps {
                sh 'mvn package'
                archiveArtifacts artifacts: 'target/*.war', fingerprint: true
            }
        }
        
        stage('Build Docker Image') {
            agent { label 'docker-agent' } 
            steps {
                sh 'docker build -t jenkins/inbound-agent:latest .' 
                sh 'docker tag jenkins/inbound-agent:latest jenkins/inbound-agent:latest' 
            }
        }
        
        stage('Push Docker Image') {
            agent { label 'docker-agent' }  
            steps {
                sh 'echo $DOCKER_HUB_CREDS_PSW | docker login -u $DOCKER_HUB_CREDS_USR --password-stdin'
                sh 'docker push jenkins/inbound-agent:latest'  
                sh 'docker push jenkins/inbound-agent:latest'  
            }
        }
    }

 	stage('Deploy with Ansible') {
            environment {
                ANSIBLE_HOST_KEY_CHECKING = 'False'
            }
            steps {
                sh 'cd ansible && ansible-playbook -i inventory.ini deploy-app.yml'
            }
        }
	 stage('Deploy to Kubernetes') {
            steps {
                sh 'mkdir -p ~/.kube'
                sh 'cat $KUBECONFIG > ~/.kube/config'
                sh 'cd ansible && ansible-playbook k8s-deploy.yml'
            }
        }

    }

    
    post {
        always {
            node('docker-agent') {  
                sh 'docker logout'
		 sh 'rm -f ~/.kube/config'
            }
        }
    }
}