#!/usr/bin/env groovy

def secrets = [
                [path: 'kv/kubernetes', engineVersion: 2, secretValues: [
                        [envVar: 'KUBECONFIG_CONTENT', vaultKey: 'kubeconfig'],
                        ]
                ]
            ]

pipeline {
    agent any
    environment {
        registry = "dineshba/dobby"
        registryCredential = 'dockerhub'
        dockerImage = ""
    }

    stages {

        stage('Lint') {
            steps {
                script {
                    sh "go get -u golang.org/x/lint/golint"
                    sh "make lint-all"
                }
            }
        }

        stage('Tests') {
            steps {
                script {
                    sh "make test"
                }
            }
        }   

        // stage('docker build') {
        //     steps {
        //         script {
        //             def imageTag = "$registry:latest"
        //             dockerImage = docker.build imageTag
        //         }
        //     }
        // }

        // stage('docker publish') {
        //     steps {
        //       container('docker') {
        //         script {
        //             docker.withRegistry("", registryCredential) {
        //                 dockerImage.push()
        //             }
        //         }
        //       }
        //     }
        // }

        stage('Deploy App') {
             steps {
                script {
                    withVault([vaultSecrets: secrets]) {
                        sh "echo \$KUBECONFIG_CONTENT | base64 -d > kubeconfig.conf"
                        sh "kubectl apply -f deployment.yaml --kubeconfig ./kubeconfig.conf"
                        sh "rm ./kubeconfig.conf"
                    }
                }
            }
        }
    }
}
