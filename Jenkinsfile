pipeline {
    agent none
    
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
    }
}

























