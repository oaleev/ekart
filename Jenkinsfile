pipeline {
    agent any
    tools{
        jdk  'jdk17'
        maven  'maven3'
        docker 'docker'
    }
    
    environment{
        SCANNER_HOME= tool 'sonar-scanner'
    }
    
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/raleev/ekart.git'
            }
        }
        
        stage('COMPILE') {
            steps {
                sh "mvn clean compile -DskipTests=true"
            }
        }
        
        stage('OWASP Scan') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ ', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        // Sonar Qube
        stage('Sonarqube') {
            steps {
                withSonarQubeEnv('Sonar'){
                   sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Shopping-Cart \
                   -Dsonar.java.binaries=. \
                   -Dsonar.projectKey=Shopping-Cart '''
               }
            }
        }
        
        stage('Build') {
            steps {
                sh "mvn clean package -DskipTests=true"
            }
        }
        stage('Deploy to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'e18e960b-9641-4d89-b5f5-cc6dbdfdc888',  , mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy -DskipTests=true"
                }
            }
        }
        
        
        stage('Docker Build & Push') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker', url: 'https://index.docker.io/v1/') {
                        sh "docker build -t shopping-cart -f docker/Dockerfile ."
                        sh "docker tag  shopping-cart studymi/shopping-cart:latest"
                        sh "docker push studymi/shopping-cart:latest"
                    }
                }
            }
        }
        
        
    }
}
