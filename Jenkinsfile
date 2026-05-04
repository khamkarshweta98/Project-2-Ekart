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
        stage('git checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/khamkarshweta98/Project-2-Ekart.git'
            }
        }
        stage('compile') {
            steps {
                sh "mvn compile"
            }
        }
        stage('unit tests') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }
        stage('SonarQube analysis') {
            steps {
                withSonarQubeEnv('sonar-scanner') {
                    sh """${env.SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=EKART \
                        -Dsonar.projectName=EKART \
                        -Dsonar.java.binaries=target/classes"""
                }
            }
        }
        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: "--nvdApiKey=049c4c7b-568d-4b66-bdab-4f4903bfb003",
                                odcInstallation: 'DC'
            }
        }
        stage('Build') {
            steps {
                sh "mvn package -DskipTests=true"
            }
        }
        stage('deploy to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-maven', jdk: 'jdk-17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy -DskipTests=true"
                }
            }
        }
        stage('build and Tag docker image') {
            steps {
                script {
                    sh "docker build -t shwetamk/ekart:latest -f docker/Dockerfile ."
                }
            }
        }
        stage('Push image to Hub') {
            steps {
                script {
                    sh '''
                        echo "dckr_pat_N5d7_nBhIUysaE8wu83dTMjK_pc" | docker login -u shwetamk --password-stdin
                        docker push shwetamk/ekart:latest
                    '''
                }
            }
        }
        stage('EKS and Kubectl configuration') {
            steps {
                script {
                    sh 'aws eks update-kubeconfig --region ap-south-1 --name project-cluster'
                }
            }
        }
        stage('Deploy to k8s') {
            steps {
                script {
                    sh 'kubectl apply -f deploymentservice.yml'
                }
            }
        }
    }
}
