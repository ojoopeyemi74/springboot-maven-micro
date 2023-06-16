pipeline {
    agent any

    tools {
        maven "maven"
    }

    environment {
        registry = '772745136297.dkr.ecr.eu-west-2.amazonaws.com/hellodatarepo'
        registryCredential = 'jenkins-ecr-login-credentials'
        dockerImage = ''
    }

    stages {
        stage("Checkout from GitHub") {
            steps {
                git branch: 'master', url: 'https://github.com/ojoopeyemi74/springboot-maven-micro.git'
            }
        }

        stage("Clean and Test") {
            steps {
                sh 'mvn clean test'
            }
        }

        stage("Maven Package") {
            steps {
                sh 'mvn package'
            }
        }

        stage("SonarQube Analysis") {
            steps {
                withSonarQubeEnv(installationName: 'SonarQube', credentialsId: 'jenkins-sonar-token') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage("Quality Gate") {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Image') {
            steps {
                script {
                    dockerImage = docker.build("${registry}:${BUILD_NUMBER}")
                }
            }
        }

        stage('Push Image to Registry') {
            steps {
                script {
                    docker.withRegistry("http://${registry}", "ecr:eu-west-2:${registryCredential}") {
                        dockerImage.push()
                    }
                }
            }
        }
    }

    post {
        success {
            mail bcc: '', body: 'Pipeline build successfully', cc: '', from: 'ojo.opeyemi747474@gmail.com', replyTo: '', subject: 'The pipeline is successful', to: 'ojo.opeyemi747474@gmail.com'
        }
        failure {
            mail bcc: '', body: 'Pipeline build failed', cc: '', from: 'ojo.opeyemi747474@gmail.com', replyTo: '', subject: 'The pipeline failed', to: 'ojo.opeyemi747474@gmail.com'
        }
    }
}
