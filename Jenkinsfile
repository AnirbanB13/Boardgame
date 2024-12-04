pipeline {
    agent any
    tools{
        jdk 'jdk17'
        maven 'maven3'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }

    stages {
        stage('Git checkout') {
            steps {
                git branch: 'main', credentialsId: 'git-token', url: 'https://github.com/Nithya-95/Boardgame.git'
            }
        }
        stage('compile') {
            steps {
               sh "mvn compile"
            }
        }
        stage('Test') {
            steps {
               sh "mvn test"
            }
        }
        stage('file system scan') {
            steps {
               sh "trivy fs --format table -o trivy-fs-output.html"
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                  sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.ProjectName=Boardgame -Dsonar.Projectkey=Boardgame \
                  -Dsonar.Java.binaries=. '''
                  }
                }
        }
        stage('Qualitygate Analysis') {
            steps {
                script{
               waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
                }
        }
        stage('Build') {
            steps {
               sh "mvn package"
            }
        }
        stage('Publish Artifacts to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'Globalsettings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                 sh "mvn deploy"
               }
            }
        }
        stage('Build Docker image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-token', toolName: 'docker') {
                      sh "docker build -t nithya95/boardgame:latest ."
                   }
                }
            }
        }
        stage('Docker Image scan') {
            steps {
               sh "trivy image --format table -o trivy-fs-output.html nithya95/boardgame:latest"
            }
        }
         stage('Push Docker image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-token', toolName: 'docker') {
                      sh "docker push nithya95/boardgame:latest"
                  }
                }
            }
        }
        stage('Deploy to kubernetes') {
            steps {
              withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'kube-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.25.160:6443') {
                sh "kubectl apply -f deployment-service.yaml"
                }
            }
        }
        
        stage('Verify the Deployment') {
            steps {
              withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'kube-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.25.160:6443') {
                sh "kubectl get pods"
                sh "kubectl get svc"
                }
            }
        }
    }
}
