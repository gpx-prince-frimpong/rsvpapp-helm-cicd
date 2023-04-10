pipeline {
    agent any

    options {
        buildDiscarder(logRotator(numToKeepStr: '30', artifactNumToKeepStr: '30'))
    }
    environment {
        HELM_REPOSITORY = 'https://nexus.payex.live/repository/helm/'
        HELM_REPOSITORY_CREDENTIALS = 'pk-nexus'
        HELM_VERSION = 'v3.9.2'
    }

    stages {
        stage('Install Build Tools'){
            steps {
                sh "wget https://get.helm.sh/helm-${HELM_VERSION}-linux-amd64.tar.gz"
                sh "tar -xvzf helm-${HELM_VERSION}-linux-amd64.tar.gz"
                sh 'sudo cp linux-amd64/helm /usr/bin'
                sh 'helm version'
            }
        }
        stage('Cleanup Artifacts'){
            steps {
                sh 'rm -rf *.tgz'
            }
        }
        stage('Checkout SCM') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: 'main']], extensions: [], userRemoteConfigs: [[credentialsId: 'payex', url: 'https://github.com/gpx-prince-frimpong/rsvpapp-helm-cicd.git']]])
            }
        }
        stage('Build Packges') {
            parallel {
                stage('Test App') {
                    steps {
                        sh "helm package rsvpapp"
                    }
                }
            }
        }
        stage('Push To Repo') {
            parallel {
                stage('Test App') {
                    steps {
                        withCredentials([usernamePassword(credentialsId: HELM_REPOSITORY_CREDENTIALS, passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                            sh "curl -u $USERNAME:$PASSWORD ${HELM_REPOSITORY} --upload-file rsvpapp-*.tgz"
                        }
                    }
                }
            }
        }
    }
}
