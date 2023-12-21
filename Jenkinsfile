// Jenkinsfile by Lawrence
pipeline {
    agent any

    stages {
        stage('Git checkout') {
            steps{
                // Get source code from a GitHub repository
                git branch:'main', url:'https://github.com/liwenbo55/p3_Techscrum.fe.git'
            }
        }
        
        stage('ci'){
            steps{
                sh 'node -v'
                sh 'yarn -v'
                sh 'cp .env.example .env'
                sh 'yarn --force'
                sh 'yarn run build'
            }
        }

        stage('Deploy to AWS'){
            steps {
                  withAWS(region:'ap-southeast-2',credentials:'lawrence-jenkins-credential') {
                      // sh 'echo "Uploading artifacts with AWS creds"'
                      sh 'aws s3 sync ./build s3://p3.techscrum-uat.wenboli.xyz-frontend-uat'
                      s3Upload(file:'./build', bucket:'p3.techscrum-uat.wenboli.xyz-frontend-uat')
                  }
            }
        }
        
    post {
        success {
            emailext(
                to: "lawrence.wenboli@gmail.com",
                subject: "Jenkins Pipeline succeeded.",
                body: "Your Jenkins Pipeline succeeded.",
                attachLog: false
            )
        }

        failure {
            emailext(
                to: "lawrence.wenboli@gmail.com",
                subject: "Jenkins Pipeline failed.",
                body: "Your Jenkins Pipeline failed，please check logfile for more details.",
                attachLog: true
            )
        }
    }
    }
}
