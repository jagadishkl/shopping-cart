pipeline {
    agent any
    
    tools{
        jdk 'OpenJDK-11'
        maven 'maven3'
    }
    environment{
        SCANNER_HOME= tool 'sonarqube-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', changelog: false, credentialsId: 'jenkinscred', poll: false, url: 'https://github.com/jagadishkl/shopping-cart.git'
            }
        }
        stage('Compile') {
            steps {
                sh "mvn clean compile -DskipTests=true"
            }
        }
        stage('OWASP Scan') {
            steps {
                //dependencyCheck additionalArguments: '--scan ./ --format HTML', odcInstallation: 'dependency-check'
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'dependency-check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('SonarQube') {
            steps {
                withSonarQubeEnv('sonar-server'){
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=shopping-cart \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=shopping-cart'''
                }
            }
        }
        stage('Build') {
            steps {
                sh "mvn clean package -DskipTests=true"
            }
        }
        stage('Docker Build & Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', url: 'https://index.docker.io/v1/') {
                        sh "docker build -t kljagadish/shopping-cart:latest -f docker/Dockerfile ."
                        sh "docker push kljagadish/shopping-cart:latest"
                    }
                }
            }
        }
    }
}
