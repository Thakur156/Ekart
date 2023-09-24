pipeline {
    agent any
    
    tools {
        jdk 'jdk17'
        maven 'maven3.6'
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('git checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Thakur156/Ekart'
            }
        }
        
        stage('compile') {
            steps {
                sh "mvn compile"
            }
        }
        
        stage('Build') {
            steps {
                sh "mvn package -DskipTests=true"
            }
        }
        
        stage('OWASP Scan') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ ', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Trivy') {
            steps {
                sh "trivy fs ."
            }
        }
        
        stage('SONARQUBE ANALYSIS') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh """$SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=Shopping-Cart \
                        -Dsonar.java.binaries=/var/lib/jenkins/workspace/E-kart/target/classes/com \
                        -Dsonar.projectKey=Shopping-Cart
                    """
                }
            }
        }
        
        stage('Deploy to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-setting') {
                    sh "mvn deploy -DskipTests=True"
                }
            }
        }
        
        stage('Build and push Docker Image') {
            steps {
                withDockerRegistry(credentialsId: 'docker-cred', url: 'https://index.docker.io/v1/') {
                    sh "docker build -t shopping-cart -f docker/Dockerfile ."
                    sh "docker tag shopping-cart thakur156/shopping-cart:latest"
                    sh "docker push thakur156/shopping-cart:latest"
                }
            }
        }
        
        stage('Run Container') {
            steps {
                withDockerRegistry(credentialsId: 'docker-cred', url: 'https://index.docker.io/v1/') {
                    sh "docker run -d --name ekart -p 8070:8070 thakur156/shopping-cart:latest "
                }
            }
        }
    }
}
