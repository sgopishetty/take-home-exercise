parameters {
   string(name: 'git_commit_hash', defaultValue: 'default', description: 'Git Commit Hash')
}

pipeline {
    environment {
        registry = "bnikkhil14/take-home-exercise"
        registryCredential = "dockerhub"
        dockerImage = ""
    }
    agent any
    tools { 
        maven 'mvn361' 
        jdk 'jdk8' 
    }
    stages {
        stage ('Initialize') {
            steps {
                sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
                ''' 
            }
        }
        
        stage('Clone repository') {
           steps {
              checkout([$class: 'GitSCM', branches: [[name: '${params.git_commit_hash}']], 
                        userRemoteConfigs: [[credentialsId: 'github', url: 'https://github.com/sgopishetty/take-home-exercise.git']]])
           }

        stage ('Code Build') {
            steps {
                sh 'mvn -Dmaven.test.failure.ignore=true package'
            }
            post {
                success {
                    junit 'target/surefire-reports/**/*.xml' 
                }
            }
        }
        stage ('Docker Build') {
            steps {
                script {
                    dockerImage = docker.build (registry + ":${BUILD_NUMBER}", "--build-arg JARFILE=person-0.0.1-SNAPSHOT.jar .")
                }
            }
        }
        stage ('Docker Publish') {
            steps {
                script {
                    docker.withRegistry('', registryCredential) {
                        dockerImage.push()
                    }
                }
            }
        }
        stage('Remove Unused docker image') {
          steps{
            sh "docker rmi $registry:$BUILD_NUMBER"
          }
        }
    }
}
