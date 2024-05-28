pipeline {
 agent { node { label "maven-sonarqube-deploy-node" } }
 parameters   {
   choice(name: 'aws_account',choices: ['590183905657'], description: 'aws account hosting image registry')
   choice(name: 'ecr_tag',choices: ['1.1.0','1.2.0','1.3.0'], description: 'Choose the ecr tag version for the build')
       }
tools {
    maven "Maven-3.9.6"
    }
    stages {
      stage('1. Git Checkout') {
        steps {
          git branch: 'main', credentialsId: 'PAT', url: 'https://github.com/Dominionsys-Inc/addressbook.git'
        }
      }
      stage('2. Build with maven') { 
        steps{
          sh "mvn clean package"
         }
       }
      stage('3. SonarQube analysis') {
      environment {SONAR_TOKEN = credentials('Team-A-Address-book-Deployment')}
      steps {
       script {
         def scannerHome = tool 'SonarQube_Scanner-5.0.1';
         withSonarQubeEnv("Team-A-Address-book-Deployment") {
         sh "${tool("SonarQube_Scanner-5.0.1")}/bin/sonar-scanner -X \
           -Dsonar.projectKey=Team-A-Address-book-Deployment \
           -Dsonar.projectName='Team-A-Address book-Deployment' \
           -Dsonar.host.url=https://172.31.39.4:9000 \
           -Dsonar.token=$SONAR_TOKEN \
           -Dsonar.sources=src/main/java/ \
           -Dsonar.java.binaries=target/classes" 
          }
         }
       }
      }
      stage('4. Docker image build') {
         steps{
          sh "aws ecr get-login-password --region us-west-2 | sudo docker login --username AWS --password-stdin ${params.aws_account}.dkr.ecr.us-west-2.amazonaws.com"
          sh "sudo docker build -t addressbook ."
          sh "sudo docker tag addressbook:latest ${params.aws_account}.dkr.ecr.us-west-2.amazonaws.com/addressbook:${params.ecr_tag}"
          sh "sudo docker push ${params.aws_account}.dkr.ecr.us-west-2.amazonaws.com/addressbook:${params.ecr_tag}"
         }
       }
      stage('5. Deployment into kubernetes cluster') {
        steps{
          kubeconfig(caCertificate: '',credentialsId: 'k8s-kubeconfig', serverUrl: '') {
          sh "kubectl apply -f manifest"
          }
         }
       }

      stage ('6. Email Notification') {
         steps{
         mail bcc: 'mbinuintangku@gmail.com', body: '''Build is Over. Check the application using the URL below. 
         https//abook.shiawslab.com/addressbook-1.0
         Let me know if the changes look okay.
         Thanks,
         Dominion System Technologies,
         +1 (313) 413-1477''', cc: 'mbinuintangku@gmail.com', from: '', replyTo: '', subject: 'Application was Successfully Deployed!!', to: 'mbinuintangku@gmail.com'
      }
    }
 }
}



