```
pipeline {
    agent any
    
    tools{
        maven 'maven3'
    }

    stages {
        
        stage('Git Checkout') {
            steps {
                git branch: 'main', credentialsId: 'git-cred',url: 'https://github.com/youssef035/maven-tutorial.git'
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
        
        
        stage('build') {
            steps {
              sh  'mvn package'
            }
        }
        
        stage('build docker image'){
            steps {
                sh 'docker build -t youssef035/mavenApp:latest'
            }
        }
      
    }
}

```
