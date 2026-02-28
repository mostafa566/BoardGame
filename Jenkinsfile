pipeline {
    agent any
    tools {
        jdk 'jdk1'
        maven 'maven11'
    }
    environment  {
        SCANNER= tool 'sonarqubescan'
    }

    stages {
        stage('Git checkout') {
            steps {
                git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/mostafa566/BoardGame.git'
            }
        }
        stage('compile') {
            steps {
                sh "mvn compile"
            }
        }
        stage('test') {
            steps {
                sh "mvn test"
            }
        }
        stage('file system scan') {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarserver') {
                    sh ''' $SCANNER/bin/sonar-scanner -Dsonar.projectName=BoardGame -Dsonar.projectKey=BoardGame \
                            -Dsonar.java.binaries=. '''
                }
            }
        }
        stage('Quality Gate') {
            steps {
                script {
                  waitForQualityGate abortPipeline: true, credentialsId: 'token' 
                }
            }
        }
        
        stage('publish to nexus repo') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global', jdk: 'jdk1', maven: 'maven11', traceability: true) {
                    sh "mvn deploy"
                }
            }
        }
        stage('build docker image:tag') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'dockerhub-cred') {
                        sh " docker build -t moustapha11/mine:latest ."
                    }
                }
            }
        }
        stage('Trivy scan image') {
            steps {
                sh "trivy image  --format table -o trivy-fs-report.html moustapha11/mine:latest"
            }
        }
        stage('push docker image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'dockerhub-cred') {
                        sh " docker push  moustapha11/mine:latest "
                    }
                }
            }
        }
        stage('Deploy to K8s ') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8s-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://44.222.145.152:6443') {
                    sh "kubectl apply -f deployment-service.yaml"
                }
            }
        }
        stage('Verfying deployment: K8s ') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8s-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://44.222.145.152:6443') {
                    sh "kubectl get pods -n webapps "
                }
            }
        }
        

    }
}
