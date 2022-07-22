pipeline {
    // 스테이지 별로 다른 거11
    agent any

    triggers {
        pollSCM('*/3 * * * *')
    }

    // 환겨 정보
    environment {
      AWS_ACCESS_KEY_ID = credentials('awsAccessKeyId')
      AWS_SECRET_ACCESS_KEY = credentials('awsSecretAccessKey')
      AWS_DEFAULT_REGION = 'ap-northeast-2'
      HOME = '.' // Avoid npm root owned
    }

    // stages : 파이프라인 stage: 작업 들..
    stages {
        // 레포지토리를 다운로드 받음
        stage('Prepare') {
            agent any
            // git에서 clone 해오기?
            steps {
                echo 'Clonning Repository'

                git url: 'https://github.com/jjang620/jenkins_test.git',
                    branch: 'master',
                    credentialsId: 'jenkins_test'
            }

            post {
                // If Maven was able to run the tests, even if some of the test
                // failed, record the test results and archive the jar file.
                // 성공했을 때
                success {
                    echo 'Successfully Cloned Repository'
                }
                // 성공하던 실패하던 띄우는
                always {
                  echo "i tried..."
                }
                // 모든 post 작업이 끝나고 나서 띄우는
                cleanup {
                  echo "after all other post condition"
                }
            }
        }
        
        // aws s3 에 파일을 올림
        stage('Deploy Frontend') {
          steps {
            echo 'Deploying Frontend'
            // 프론트엔드 디렉토리의 정적파일들을 S3 에 올림, 이 전에 반드시 EC2 instance profile 을 등록해야함.
            dir ('./website'){
                sh '''
                aws s3 sync ./ s3://jhw620/jenkins
                '''
            }
          }

          post {
              // If Maven was able to run the tests, even if some of the test
              // failed, record the test results and archive the jar file.
              success {
                  echo 'Successfully Cloned Repository'

                  mail  to: 'ehrtk2003@gmail.com',
                        subject: "Deploy Frontend Success",
                        body: "Successfully deployed frontend!"

              }

              failure {
                  echo 'I failed :('

                  mail  to: 'ehrtk2003@gmail.com',
                        subject: "Failed Pipelinee",
                        body: "Something is wrong with deploy frontend"
              }
          }
        }
        
        stage('Lint Backend') {
            // Docker plugin and Docker Pipeline 두개를 깔아야 사용가능!
            agent {
              docker {
                image 'node:latest'
              }
            }
            
            steps {
              dir ('./server'){
                  sh '''
                  npm install&&
                  npm run lint
                  '''
              }
            }
        }
        
        stage('Test Backend') {
          agent {
            docker {
              image 'node:latest'
            }
          }
          steps {
            echo 'Test Backend'

            dir ('./server'){
                sh '''
                npm install
                npm run test
                '''
            }
          }
        }
        
        stage('Bulid Backend') {
          agent any
          steps {
            echo 'Build Backend'

            dir ('./server'){
                sh """
                docker build . -t server --build-arg env=${PROD}
                """
            }
          }

          post {
            // 위 스테이지에선 실패해도 다음 작업으로 넘어갔느데 빌드를  실패했느데 계속 진행되면 안되니 여기선 강제 종료하도록 함.
            failure {
              error 'This pipeline stops here...'
            }
          }
        }
        
        stage('Deploy Backend') {
          agent any

          steps {
            echo 'Build Backend'

            dir ('./server'){
                // docker rm -f $(docker ps -aq) 첫번째 실행할땐 제거 두번째 부터 아래 sh에 넣어준다.
                sh '''         
                docker run -p 80:80 -d server
                '''
            }
          }

          post {
            success {
              mail  to: 'frontalnh@gmail.com',
                    subject: "Deploy Success",
                    body: "Successfully deployed!"
                  
            }
          }
        }
    }
}
