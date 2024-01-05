// Jenkinsfile for Techscrum project (by Lawrence)
pipeline {
    agent any

    parameters {
        // Choose an environment to deploy frond-end.
        choice(choices: ['dev', 'uat', 'prod'], name: 'Environment', description: 'Please choose an environment to deploy frond-end.')
    }
    
    environment {
        DEV_DISTRIBUTION_ID  = ""
        UAT_DISTRIBUTION_ID  = credentials('UAT_DISTRIBUTION_ID')
        PROD_DISTRIBUTION_ID = ""
        S3_BUCKET = "p3.techscrum-${params.Environment}.wenboli.xyz-frontend-${params.Environment}"
    }
    

    stages {
        stage('Git checkout') {
            steps{
                // Get source code from a GitHub repository
                git branch:'test/jenkins-pipeline', url:'https://github.com/liwenbo55/p3_Techscrum.fe.git'
            }
        }
        
        stage('ci'){
            steps{
                script{
                    sh 'test test test.....'
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
                }
            }
        }

        stage('cd'){
            steps {
                script{
                    withAWS(region:'ap-southeast-2',credentials:'lawrence-jenkins-credential') {
                     // s3Upload(file:'./build', bucket:'p3.techscrum-uat.wenboli.xyz-frontend-uat')
                      sh 'aws s3 sync ./build s3://${S3_BUCKET}'
                      // Either is OK to upload artifacts to S3 bucket

                      // CLoudfront invalidation
                      if (params.Environment == 'dev'){
                          sh 'aws cloudfront create-invalidation --distribution-id ${DEV_DISTRIBUTION_ID} --paths "/**/*"'
                      } else if (params.Environment == 'uat'){
                          sh 'aws cloudfront create-invalidation --distribution-id ${UAT_DISTRIBUTION_ID} --paths "/**/*"'
                      } else if (params.Environment == 'prod'){
                          sh 'aws cloudfront create-invalidation --distribution-id ${PROD_DISTRIBUTION_ID} --paths "/**/*"'
                      } else {
                          error "Invalid environment: ${params.Environment}."
                      }
                    }
                }
            }
        }
    }
        
    post {
        success {
            echo "Frontend URL: ${params.Environment}.wenboli.xyz"
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
                body: "Jenkins Pipeline failed, please check logfile for more details.",
                attachLog: true
            )
        }
    }
}
