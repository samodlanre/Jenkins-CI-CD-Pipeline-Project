def COLOR_MAP = [
    'SUCCESS': 'good', 
    'FAILURE': 'danger',
]




pipeline {
    agent any
    
    environment{
       
        WORKSPACE = "${env.WORKSPACE}"
       
    }
    
    
    tools{
        maven 'localMaven'
        jdk 'localJdk'
    }

    stages {
        stage('Git checkout') {
            steps {
                echo 'Cloning the application code...'
                git branch: 'main', url:'https://github.com/samodlanre/Jenkins-CI-CD-Pipeline-Project.git'
                sh 'mvn --version'
                
            }
        }
        
        stage('Build') {
            steps {
                sh 'java -version'
                
                 sh 'mvn clean package'
                
                
            }
        post {
            success {
                echo 'archiving.....'
                archiveArtifacts artifacts: '**/*.war', followSymlinks: false
            }
        }
        
            
            
            
        }
        
        stage('Unit Test'){
        steps {
            sh 'mvn test'
        }
    }
    stage('Integration Test'){
        steps {
          sh 'mvn verify -DskipUnitTests'
        }
    }
    stage ('Checkstyle Code Analysis'){
        steps {
            sh 'mvn checkstyle:checkstyle'
        }
        post {
            success {
                echo 'Generated Analysis Result'
            }
        }
    }
    
    stage ('SonarQube scanning'){
        steps {
            
            withSonarQubeEnv('SonarQube') {
            sh """
            mvn sonar:sonar \
  -Dsonar.projectKey=JavaWebApp \
  -Dsonar.host.url=http://172.31.95.248:9000 \
  -Dsonar.login=fef5295bcb88b5d6c8d28326ca7afd64c11429ca
            
            
            """
            
        }
       
    }
    
    }
    
    stage("Quality Gate"){
    
  steps{
       
   waitForQualityGate abortPipeline: true
       
     }
        
    }
    
    stage("Upload artifact to nexus"){
    
  steps{
       
   sh 'mvn clean deploy -DskipTest'
       
     }
        
    }
    
    stage('Deploy to DEV') {
      environment {
        HOSTS = "dev"
      }
      steps {
        sh 'ls'
        sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
      }
     }
     
     
    
   
    
    
    stage('Deploy to STAGE env') {
      environment {
        HOSTS = "stage"
      }
        steps {
        sh 'ls'
        sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
      }
     }
      
       stage('Approval') {
      steps {
        input('Do you want to proceed?')
      }
    }
    
     stage('Deploy to PROD env') {
      environment {
        HOSTS = "prod"
      }
      steps {
        sh 'ls'
        sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
      }
     }
        
        
        
        
        
        
    }
    post { 
        always { 
            echo 'I will always say Hello again!'
            slackSend channel: '#cicd', color: COLOR_MAP[currentBuild.currentResult], message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
        }
    }
}