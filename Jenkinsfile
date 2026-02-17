pipeline {
    agent any
     tools{
         jdk 'jdk17'
         maven 'mvn3'
     }
     environment{
         SCANNER_HOME=tool "sonar-scanner"
         IMAGE_TAG = "${BUILD_NUMBER}"
     }
     
    stages {
        stage('git checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Cloud-Lucky/Boardgame.git'
            }
        }
        
        stage('compile') {
            steps {
                sh 'mvn compile'
            }
        }
        
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        
        stage('sonar analysis') {
            steps {
                 withSonarQubeEnv('sonar'){
                 sh'''
                 "$SCANNER_HOME"/bin/sonar-scanner -Dsonar.projectKey=Bank -Dsonar.projectName=Bank
                 '''
                 }
            }
        }
        
        stage('Gateway-check') {
            steps {
                script{
                    timeout(time: 1, unit: 'HOURS'){
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                    }
                    
                }
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn package'
            }
        }
        
        stage('Deploy to nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'jenkins', jdk: 'jdk17', maven: 'mvn3', traceability: true) {
                    sh'mvn deploy'
                }
            }
        }
        
         stage('Docker login build') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'Docker-cred', passwordVariable: 'DockerHubPassword', usernameVariable: 'DockerHubUser')]){
                    sh '''
                    echo "$DockerHubPassword" | docker login -u "$DockerHubUser" --password-stdin
                     docker build -t laxmireddy870113/bank:v${IMAGE_TAG} .
                     docker push laxmireddy870113/bank:v${IMAGE_TAG}
                     '''
                }
            }
        }
        
        stage('Docker-run') {
            steps {
                sh 'docker run -d -p 8000:8080 laxmireddy870113/bank:v${IMAGE_TAG} '
            }
        }
    }
}
