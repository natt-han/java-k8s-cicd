pipeline {
    agent any
    environment {
        CI = 'true'
    }
    stages {
        stage('SonarQube analysis') {
            steps {
                withSonarQubeEnv(credentialsId: 'sonarq-java-token', installationName: 'SonarQube') { // You can override the credential to be used
                    sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.6.0.1398:sonar -Dsonar.projectKey=java-k8s-cicd -Dsonar.sources=src/main'
                }
            }
        }
        stage('Build image') { 
            steps {
                script {
                    echo "Build image with tag: ${env.BUILD_ID}"
                    dockerImage = docker.build("172.28.128.3:30700/java-k8s-cicd:${env.BUILD_ID}")
                }
            }
        }
        stage('Push image to registry') { 
            steps {
                script {
                    docker.withRegistry('http://172.28.128.3:30700', 'nexus-user-and-password') {
                        dockerImage.push()
                        dockerImage.push('latest')
                    }
                }
            }
        }
        stage('Deploy to K8s cluster') { 
            steps {
                script {
                    withCredentials([sshUserPrivateKey(credentialsId: 'vagrant-ssh', keyFileVariable: 'SSHKEY')]) {
                        sh '''                            
                            sed -i 's/latest/'"${BUILD_ID}"'/g' java-k8s-cicd-k82-deployment.yaml
							scp -o StrictHostKeyChecking=no -i ${SSHKEY} java-k8s-cicd-k82-deployment.yaml vagrant@10.32.0.1:/home/vagrant/
							ssh -i ${SSHKEY} vagrant@10.32.0.1 kubectl apply -f java-k8s-cicd-k82-deployment.yaml
                        '''
                    }
                    
                }
            }
        }
    }
}