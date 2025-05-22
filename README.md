# pipeline
````
pipeline {
    agent any 
    tools {
        maven 'maven'
    }
    
    stages{
        stage('code-checkout'){
            steps{
                git branch: 'main', url: 'https://github.com/itsdevopsprofile/jenkins-project.git'
            }
        }
        stage('code-build'){
            steps{
                sh 'mvn clean package'
            }
        }
        stage('code-deploy'){
            steps{
                 sh 'docker build -t test .'
                 sh 'docker run -itd --name cont-1 -p 8089:8081 test'
            }
        }
    }
}
````
