```groovy
pipeline {

    agent any

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    tools {
        maven 'maven3'
        jdk 'jdk-17'
    }

    stages {

        stage('Git Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Kirtika0/Ekart.git'
            }
        }

        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }

        stage('Unit Tests') {
            steps {
                sh 'mvn test -DskipTests=true'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-scanner') {
                    sh '''
                        ${SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=EKART \
                        -Dsonar.projectName=EKART \
                        -Dsonar.java.binaries=target/classes
                    '''
                }
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck(
                    odcInstallation: 'DC',
                    nvdCredentialsId: 'nvd-api-key'
                )
            }
        }

        stage('Build') {
            steps {
                sh 'mvn package -DskipTests=true'
            }
        }

        stage('Deploy to Nexus') {
            steps {
                withMaven(
                    globalMavenSettingsConfig: 'global-maven',
                    jdk: 'jdk-17',
                    maven: 'maven3',
                    mavenSettingsConfig: '',
                    traceability: true
                ) {
                    sh 'mvn deploy -DskipTests=true'
                }
            }
        }

        stage('Build and Tag Docker Image') {
            steps {
                script {
                    sh 'docker build -t kirtika0/ekart:latest -f docker/Dockerfile .'
                }
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                script {

                    withCredentials([
                        string(
                            credentialsId: 'dockerhub-pwd',
                            variable: 'dockerhubpwd'
                        )
                    ]) {

                        sh '''
                            echo "$dockerhubpwd" | docker login \
                            -u kirtika0 \
                            --password-stdin
                        '''
                    }

                    sh 'docker push kirtika0/ekart:latest'
                }
            }
        }

        stage('EKS and Kubectl Configuration') {
            steps {
                script {
                    sh '''
                        aws eks update-kubeconfig \
                        --region ap-south-1 \
                        --name project-cluster
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh 'kubectl apply -f deploymentservice.yml'
                }
            }
        }

    }
}
```
