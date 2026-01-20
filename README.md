# pipeline
````
pipeline {
    agent any
    tools {
        maven 'maven'
    }
    
    stages{
        stage('code-pull'){
            steps{
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/itsdevopsprofile/jenkins-project.git']])
            }
        }
        
        stage('code-build'){
            steps{
                sh 'mvn clean package'
            }
        }
        stage('code-deploy'){
            steps{
                sh 'docker build -t demo .'
                sh 'docker run -itd --name cont1 -p 8089:8081 demo'
            }
        }
    }
}
````


