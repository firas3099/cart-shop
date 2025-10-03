pipeline {
    agent any
    tools{
        jdk 'jdk11'
        maven 'maven3'
    }
    environment{
        SCANNER_HOME= tool 'sonar-scanner'
    }
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', changelog: false, credentialsId: '1cb00f2a-a7e8-4153-9a49-b5416d2d2456', poll: false, url: 'https://github.com/firas3099/cart-shop.git'
            }
        }
        stage('Compile') {
            steps {
            sh "mvn clean compile -DskipTests=True"
            }
        }
         stage('OWASP Scan') {
             steps {
             dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'DP'
             dependencyCheckPublisher pattern: 'dependency-check-report.xml'
             }
         }
          stage('Sonarqube ') {
            steps {
            withSonarQubeEnv('sonar-server') {
                sh ''' $SCANNER_HOME/bin/sonar-scanner  -Dsonar.projectName=Shopping-Cart \
                        -Dsonar.java.binaries=. \
                        -Dsonar.projectKey=Shopping-Cart'''
            }
            }
        }
        stage('build JAR/WAR') {
            steps {
            sh "mvn clean package -DskipTests=True"
            }
        }
        stage('build docker image'){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker_hub', toolName: 'docker') {
                        sh "docker build -t firasmhalla3099/shopping-cart -f docker/Dockerfile ."
                        sh "docker push firasmhalla3099/shopping-cart"
                    }
                }
            }
        }
        stage('docker run'){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker_hub', toolName: 'docker') {
                        
                        sh "docker run -d --name shop -p 8070:8070 firasmhalla3099/shopping-cart"
                    }
                }
            }
        }
        
    }
}
