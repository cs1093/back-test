pipeline {
    agent any

    environment {
        imagename = 'test'
        regiGit = 'https://github.com/cs1093/back-test.git'
        gitCredentialsId = credentials('gitToken')
        awsCredentialsId = credentials('AWS-Credential')
        awsRegion = 'ap-northeast-2'
        ecrRepo = '806308213817.dkr.ecr.ap-northeast-2.amazonaws.com'
    }

    stages {
        // git에서 repository clone
        stage('Prepare') {
          steps {
            echo 'Clonning Repository'
            git url: "${regiGit}", 
              branch: 'main',
              credentialsId: "${gitCredentialsId}"  // 생성한 github access token credentail id = 변수처리
            }
            post {
             success { 
               echo 'Successfully Cloned Repository'
             }
           	 failure {
               error 'This pipeline stops here...'
             }
          }
        }

        // gradle build
        stage('Bulid Gradle') {
          agent any
          steps {
            echo 'Bulid Gradle'
            dir ('.'){
                sh """
                chmod +x ./gradlew
                ./gradlew clean build
                """
            }
          }
          post {
            success { 
               echo 'Successfully Gradle!'
             }
            failure {
              error 'This pipeline stops here...'
            }
          }
        }
        
        // docker build
        stage('Bulid Docker') {
          agent any
          steps {
            echo 'Bulid Docker'
            // dockerImage = docker.build imagename
            script {
              sh"""
                docker build . -t 806308213817.dkr.ecr.ap-northeast-2.amazonaws.com/test:1.0
              """
            }
          }
          post {
            success { 
               echo 'Successfully Docker Build!'
             }
            failure {
              error 'This pipeline stops here...'
            }
          }
        }

        // docker push to ECR
        stage('Push Docker') {
          agent any
          steps {
            echo 'Push Docker'
            script {
                // ECR 관련 환경 변수 설정
                withAWS(credentials: awsCredentialsId, region: awsRegion) {
                    // ECR 리포지토리 URI 설정
                    def ecrRepo = "${ecrRepo}" + '/' + "${imagename}"
                    // Docker 이미지를 ECR에 푸시
                    sh """
                    docker tag ${imagename} ${ecrRepo}:${BUILD_NUMBER}
                    docker push ${ecrRepo}:${BUILD_NUMBER}
                    """
                }
            }
        }
          post {
            success { 
               echo 'Successfully Docker Push to AWS ECR!'
             }
            failure {
              error 'This pipeline stops here...'
            }
          }
        }
    }
}
