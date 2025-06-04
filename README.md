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

---
# Upload Artifacts to S3 bucket
### required plugins
- stage view
- maven integration
- aws credentials
- s3

### go to tools sections -> configure maven

### go to manage jenkins-> credentials ->aws access key & secret key-> create global creds
### note: make sure aws cli installed on server
````
sudo apt install unzip -y
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
````
## create pipeline
````
pipeline {
    agent any 
    tools{
        maven 'maven'
    }
    environment {
        S3_BUCKET = 'coffeeshop.co.in'
        AWS_REGION = 'ap-southeast-1'
        warFile = 'target/Insurance-0.0.1-SNAPSHOT.jar'
    }
    
    stages{
        stage('code-pull'){
            steps{
                git branch: 'main', url: 'https://github.com/itsdevopsprofile/jenkins-project.git'
            }
        }
        stage('code-build'){
            steps{
                sh 'mvn clean package'
            }
        }
        stage('upload artifacts'){
            steps{
                withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'aws-cred', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                 script {
                    
                     sh 'aws s3 cp ${warFile} s3://${S3_BUCKET}/artifacts/ --region ${AWS_REGION}'
                 }
               }
            }
        }
    }
}
````

