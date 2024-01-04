// Jenkinsfile for Techscrum project (by Lawrence)
pipeline {
    agent any
    
    environment {
        DISTRIBUTION_ID = "E14LUFS4II79ZY"
    }
    
    parameters {
        // Choose an environment to deploy frond-end.
        choice(choices: ['dev', 'uat', 'prod'], name: 'Environment', description: 'Please choose an environment to deploy frond-end.')
    }

    stages {
        stage('Git checkout') {
            steps{
                // Get source code from a GitHub repository
                git branch:'main', url:'https://github.com/liwenbo55/p3_Techscrum.fe.git'
            }
        }
        
        stage('ci'){
            steps{
                // Show version
                sh 'node -v'
                sh 'yarn -v'

                // .env file
                if (params.Environment == 'dev'){
                    sh 'cp .env.dev .env'
                } else if (params.Environment == 'uat'){
                    sh 'cp .env.uat .env'
                } else if (params.Environment == 'prod'){
                    sh 'cp .env.prod .env'
                } else {
                    error "Invalid environment: ${params.Environment}."
                }
                sh "cat .env"

                // build
                sh 'yarn --force'
                sh 'yarn run build'

                // sh 'node -v'
                // sh 'npm -v'
                // sh 'npm install --force'
                // sh 'cp .env.example .env'
                // sh 'npm run build'
            }
        }

        stage('Deploy to AWS'){
            steps {
                  withAWS(region:'ap-southeast-2',credentials:'lawrence-jenkins-credential') {
                      s3Upload(file:'./build', bucket:'p3.techscrum-${params.Environment}.wenboli.xyz-frontend-${params.Environment}')
                      // sh 'aws s3 sync ./build s3://p3.techscrum-uat.wenboli.xyz-frontend-uat'
                      // Either is OK to upload artifacts to S3 bucket
                  }
            }
        }

        stage('CLoudfront invalidation'){
            steps{
                withAWS(region:'ap-southeast-2',credentials:'lawrence-jenkins-credential') {
                    sh 'aws cloudfront create-invalidation --distribution-id ${DISTRIBUTION_ID} --paths "/**/*"'
                }
            }
        }
    }
        
    post {
        success {
            emailext(
                to: "lawrence.wenboli@gmail.com",
                subject: "Front-end cicd pipeline for ${params.Environment} environment succeeded.",
                body: "Jenkins Pipeline succeeded.\nEnvironment: ${params.Environment}.",
                attachLog: false
            )
        }

        failure {
            emailext(
                to: "lawrence.wenboli@gmail.com",
                subject: "Front-end cicd pipeline for ${params.Environment} environment failed.",
                body: "Jenkins Pipeline failed，please check logfile for more details.",
                attachLog: true
            )
        }
    }
}
