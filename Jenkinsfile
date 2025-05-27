pipeline {
    agent any
    stages {
        stage('install  os deps') {
            steps {
                sh 'apt-get update && apt-get install npm -y'
            }
        }