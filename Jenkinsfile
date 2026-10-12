pipeline {
    agent any
    tools {
        jdk 'JDK21'
        maven 'M3'
    }
    environment {
            DOCKERHUB_CRED = credentials('dockerCredentials')
    }

    stages {
        // Github로부터 소스코드 다운로드
        stage('Git Clone') {
            steps {
                echo 'Git Clone'
                git url: 'https://github.com/dlwnsgur6889-pixel/spring-petclinic.git',
                branch: 'main'
            }
        }
        //Docker Image 생성
        stage 'Docker Image Create') {
            steps {
                echo 'Docker Image Create'
                sh '''
                docker butld -t spring-petclinic:${BUILD_NUMBER} .
                docker tag spring-petclininc:${BUILD_NUMBER} krljh/spring-petclinic:latest
                '''
        }
        //Dockerhub Image Push
        stage('Docker Image Push') {
            steps {
                echo 'Docker Images Push'
                // Docker Login
                sh 'eco ${DOCKERHUB_CRED_PSW} | docker login -u ${DOCKERHUB_CRED_USR} --password-studin

                //Docker Hub Push
                sh 'docker image push krljh/spring-petclinic:latest'
            }
            post {
                always {
                    sh '''
                    docker rmi -f spring-petclinic:${BUILD_NUMBER}
                    docker rmi -f krljh/spring-petclinic:latest
                    '''
                }
            }
        }
        //Docker Image 삭제
        // Maven을 이용한 Build
        stage('Maven Build') {
            steps {
                echo 'Maven Build'
                sh 'mvn -Dmaven.test.failure.ignore=true clean package'
            }
        }
        // SSH를 이용한 실행파일 전송
        stage('SSH Publish') {
            steps {
                echo 'SSH Publish'
                sshPublisher(publishers: [sshPublisherDesc(configName: 'target', 
                transfers: [sshTransfer(cleanRemote: false, excludes: '', 
                execCommand: '''
                docker rmi -f &(docker ps -aq)
                docekr rmi -f $(docker images -q)
                docker run -itd -p 80:8080 --name=spring-petclinic s4616/spring-petclinic:latest
                ''', 
                execTimeout: 120000, 
                flatten: false, 
                makeEmptyDirs: false, 
                noDefaultExcludes: false, 
                patternSeparator: '[, ]+', 
                remoteDirectory: '', 
                remoteDirectorySDF: false, 
                removePrefix: 'target', 
                sourceFiles: 'target/*.jar')], 
                usePromotionTimestamp: false, 
                useWorkspaceInPromotion: false, verbose: false)])
            }
        }   
    }
}






