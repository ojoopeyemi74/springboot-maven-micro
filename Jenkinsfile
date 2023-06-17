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
        stage ("upload war file to nexus"){
            steps {
                script{

                    def readPomVersion = readMavenPom file: 'pom.xml'

                    def nexusRepo = readPomVersion.version.endsWith("SNAPSHOT") ? "demoapp-snapshot" : "demoapp-release"
                    nexusArtifactUploader artifacts: 
                    [
                        [artifactId: 'spring-boot-starter-parent', 
                        classifier: '',
                        file: '/var/lib/jenkins/workspace/CI-pipeline/target/springboot-maven-course-micro-svc-5.0.1-SNAPSHOT.jar', 
                        type: 'jar'
                        ]
                    ],
                    credentialsId: 'nexus-auth', 
                    groupId: 'com.cloudtechmasters', 
                    nexusUrl: '13.41.184.101:8081', 
                    nexusVersion: 'nexus3', 
                    protocol: 'http', 
                    repository: nexusRepo , 
                    version: "${readPomVersion.version}"

                }
            }
        }
    }
}
