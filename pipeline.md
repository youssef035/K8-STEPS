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
        
  //you need to have docker in Jenkins Server + Permission => chmod 666 ...        
        stage('build docker image'){
            steps {
                sh 'docker build -t youssef035/petclinic:latest .'
            }
        }
        
        
        stage('push docker image'){
            steps {
                withCredentials([usernamePassword(
                    credentialsId:'docker-cred',
                    usernameVariable:'DOCKER_USER',
                    passwordVariable:'DOCKER_PASS'
                    )]){
                        sh'''docker login -u $DOCKER_USER -p $DOCKER_PASS
                             docker push youssef035/petclinic:latest
                        '''
                    }
            }
        }
        
        stage('Deploy to kubernetes'){
            steps{
                withKubeConfig(
                    credentialsId:'k8-cred',
                    clusterName:'kubernetes',
                    contextName:'',
                    namespace:'webapps',
                    restrictKubeConfigAccess:false,
                    serverUrl:'https://172.234.176.135:6443'  // change the p address with the master node of your own cluster
                    ){
                        sh'kubectl apply -f deployment-service.yml'
                    }
            }
        }

        
      
    }
}

```
