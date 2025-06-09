```
pipeline {
    agent any
    
    tools{
        maven 'maven3'
    }
    
    environment {
        SCANNER_HOME= tool'sonar-scanner'
    }

    stages {
        
        stage('Git Checkout') {
            steps {
                git branch: 'main', credentialsId: 'git-cred',url: 'https://github.com/youssef035/Petclinic.git'
            }
        }
        
        stage('compile') {
            steps {
               sh  'mvn compile'
            }
        }
        
        stage('test') {
            steps {
              sh  'mvn test'
            }
        }
        
        stage('code quality check') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh'''${SCANNER_HOME}/bin/sonar-scanner \
                    -Dsonar.projectName=Petclinic \
                    -Dsonar.projectKey=Petclinic \
                    -Dsonar.java.binaries=target/classes'''
                }
            }
        }
        
        
        stage('build') {
            steps {
              sh  'mvn package'
            }
        }

        stage('Owasp Dependency Check'){
            steps {
                dependencyCheck additionalArguments: '--scan target/', odcInstallation: 'owasp'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Push to Nexus Repository'){
            steps{
                configFileProvider([configFile(fileId: 'b12b8d19-565e-46e8-98b0-d87f2cfc08ac', variable: 'mavensettings')]) {
                    sh"mvn -s $mavensettings clean deploy -DskipTests=true"
                }
            }
        }
        
        stage('build docker image'){
            steps {
                sh 'docker build -t youssef035/mavenapp:latest .'
            }
        }
        
      
    }
}

```
