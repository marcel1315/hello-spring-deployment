pipeline {
    agent any

    tools {
        gradle 'gradle-8.10.2'
        jdk 'jdk-21'
    }

    parameters {
        choice(choices: ['dev', 'qa', 'prod'], name: 'PROFILE')
    }

    environment {
        DOCKERHUB_USERNAME = 'marcel1315'
        DEPLOY_GITHUB = 'https://github.com/marcel1315/hello-spring-deployment.git'
        SOURCE_GITHUB = 'https://github.com/marcel1315/hello-spring.git'
    }

    stages {
        stage('Source code checkout') {
            steps {
                dir('source-code') {
                    git branch: 'main', url: SOURCE_GITHUB
                }
            }
        }

        stage('Check Java Version') {
            steps {
                sh 'java -version'
                sh 'echo $JAVA_HOME'
                sh 'gradle -q javaToolchains'
            }
        }

        stage('Source build') {
            steps {
                dir('source-code') {
                    // Give 755 (644 when uploading from Windows)
                    sh "chmod +x ./gradlew"                
                    sh 'pwd'
                    sh 'ls'
                    sh "./gradlew clean build --stacktrace"                
                }
            }
        }

        stage('Releasefile checkout') {
            steps {
                dir('release-file') {
                    git branch: 'main', url: DEPLOY_GITHUB
                }
            }
        }

        stage('Container build') {
            steps {
                dir('source-code') {
                    // Copy jar
                    sh "cp ./build/libs/hello-spring-0.0.1-SNAPSHOT.jar ./build/docker/app-0.0.1-SNAPSHOT.jar"

                    // Docker build
                    sh "docker build -t ${DOCKERHUB_USERNAME}/hello-spring:1.0.0 ./build/docker"
                }
            }
        }

        stage('Container upload') {
            steps {
                dir('source-code') {
                    sh "docker push ${DOCKERHUB_USERNAME}/hello-spring:1.0.0"
                }
            }
        }

        stage('Check kustomize template') {
            steps {
                dir('release-file') {
                    sh "kubectl kustomize ./deploy/kustomize/hello-spring/overlays/${params.PROFILE}"
                }
            }
        }

        stage('Deploy kustomize') {
            steps {
                dir('release-file') {
                    input message: 'Start deployment', ok: "Yes"
                    sh "kubectl apply -f ./deploy/kubectl/namespace-${params.PROFILE}.yaml"
                    sh "kubectl apply -k ./deploy/kustomize/hello-spring/overlays/${params.PROFILE}"
                }
            }
        }
    }
}