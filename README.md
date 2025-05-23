# install required plugins
- stage view
- maven integration
- docker (first 4 plugins)


# go to tools sections
- add maven
- add docker

# go to credentials section
- add dockerhub username password

# create pipeline 


# simple pipeline
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
# pipeline with docker

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
        stage('Containerize the application'){
            steps { 
               echo 'Creating Docker image'
               sh "docker build -t abhipraydh96/insure:v12 ."
            }
        }
        
        stage('Docker Push') {
    	agent any
          steps {
       	withCredentials([usernamePassword(credentialsId: 'docker-cred', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
            	sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPassword}"
                sh 'docker push abhipraydh96/insure:v12'
           }
         }
      }
            stage('Deploy the application'){
            steps { 
               echo 'Creating Docker Cont'
               sh "docker run -itd -p 8089:8081 abhipraydh96/insure:v12 "
            }
        }
        
    }
}
````

# run pipeline 
![image](https://github.com/user-attachments/assets/13e87173-1a09-4a64-acaa-231dd22ef5d4)

# final output
![image](https://github.com/user-attachments/assets/dc3396b9-1566-4a62-be09-3ba5d6a18051)



