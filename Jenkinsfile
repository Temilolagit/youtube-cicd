pipeline {
    agent any
    tools {
        nodejs 'node21'
    }
    environment {
        DOCKER_CREDENTIALS = credentials('d4506f04-b98c-47db-95ce-018ceac27ba6')
        SCANNER_HOME = tool 'sonar-scanner'
        IMAGE_NAME = 'idrisniyi94/youtube-cicd'
    }
    stages {
        stage('Clean workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/stwins60/youtube-cicd.git']])
            }
        }
        stage('SonarQube analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh "${tool 'sonar-scanner'}/bin/sonar-scanner -Dsonar.projectKey=youtube-cicd -Dsonar.sources=."
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
                odcInstallation: 'DP-Check'
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
        stage('Docker Push') {
            steps {
                sh "docker push $IMAGE_NAME"
            }
        }
        stage('Trivy Image Scan') {
            steps {
                sh "trivy image $IMAGE_NAME"
            }
        }
    }
}