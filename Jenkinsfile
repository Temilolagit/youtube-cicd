pipeline {
    agent any
    tools {
        nodejs 'NodeJS'
    }
    environment {
        DOCKER_CREDENTIALS = credentials('Docker_hub')
        SCANNER_HOME = tool 'sonar-scanner'
        IMAGE_NAME = 'temiloladocker/youtube-cicd'
    }
    stages {
        stage('Clean workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout') {
            steps {
               git branch: 'main', url: 'https://github.com/Temilolagit/youtube-cicd.git'
            }
        }
        stage('SonarQube analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh "${tool 'sonar-scanner'}/bin/sonar-scanner -Dsonar.projectKey=Youtube -Dsonar.sources=."
                }
            }
        }
        stage('Build') {
            steps {
                sh 'npm install'
            }
        }
        stage('Trivy File Scan') {
            steps {
                sh "trivy fs ."
            }
        }
        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit --nvdApikey 4bdf4acc-8eae-45c1-bfc4-844d549be812',
                odcInstallation: 'DP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS, passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh "docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD"
                }
            }
        }
        stage('Docker Build') {
            steps {
                sh "docker build -t $IMAGE_NAME ."
            }
        }
        
        stage('Trivy Image Scan') {
            steps {
                sh "trivy image $IMAGE_NAME"
            }
        }
        stage('Docker Push') {
            steps {
                sh "docker push $IMAGE_NAME"
            }
        }
        stage('Containerization Deployment') {
            steps {
                script {
                    def containername = 'reddit-clone-k8s'
                    def isRunning = sh(script: "docker ps -a | grep ${containername}", returnStatus: true)
                    if (isRunning == 0) {
                        sh "docker stop ${containername}"
                        sh "docker rm ${containername}"
                    }
                    sh "docker run -d -p 3000:3000 --name ${containername} temiloladocker/youtube-cicd:latest"
                }                    
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig([credentialsId: 'kubeconfig']) {
                

                    sh 'kubectl apply -f deployment.yml'
                    sh 'kubectl apply -f service.yml'
                    // Wait for deployment to finish
                    //sh 'kubectl rollout status -f path/to/your/deployment.yaml'
                }
            } 
        }           
    }
}