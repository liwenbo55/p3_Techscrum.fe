// Jenkinsfile for Techscrum project (by Lawrence)
pipeline {
    agent any

    parameters {
        // Choose an environment to deploy frond-end.
        choice(choices: ['dev', 'uat', 'prod'], name: 'Environment', description: 'Please choose an environment to deploy frond-end.')
    }
    
    environment {
        DEV_DISTRIBUTION_ID  = credentials('DEV_DISTRIBUTION_ID')
        UAT_DISTRIBUTION_ID  = credentials('UAT_DISTRIBUTION_ID')
        PROD_DISTRIBUTION_ID = credentials('PROD_DISTRIBUTION_ID')
        // HOSTED_ZONE_NAME = "wenboli.xyz"
        HOSTED_ZONE_NAME = credentials('HOSTED_ZONE_NAME')
        S3_BUCKET = "p3.techscrum-${params.Environment}.${HOSTED_ZONE_NAME}-frontend-${params.Environment}"
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
                script{
                    sh "echo 'test test test.....'"
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
                     // s3Upload(file:'./build', bucket:'p3.techscrum-uat..xyz-frontend-uat')
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
            echo "Front-end URL: ${params.Environment}.${HOSTED_ZONE_NAME}"
            emailext(
                to: "lawrence.wenboli@gmail.com",
                subject: "Front-end CICD pipeline (${params.Environment} environment) succeeded.",
                body: "Jenkins CICD Pipeline succeeded.\nEnvironment: ${params.Environment}.",
                attachLog: false
            )
        }

        failure {
            emailext(
                to: "lawrence.wenboli@gmail.com",
                subject: "Front-end CICD pipeline (${params.Environment} environment) failed.",
                body: "Jenkins CICD Pipeline failed, please check logfile for more details.",
                attachLog: true
            )
        }
    }
}
