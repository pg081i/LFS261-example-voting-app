pipeline {

    agent none

    stages {

        stage('monopipe-worker-build') {

            agent {
                docker {
                    image 'maven:3.8.5-jdk-11-slim'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }

            when {
                changeset '**/worker/**'
            }

            steps {

                echo 'MONOPIPE: Compiling worker app..'

                dir(path: 'worker') {
                    sh 'mvn compile'
                }
            }
        }

        stage('monopipe-worker-test') {

            agent {
                docker {
                    image 'maven:3.8.5-jdk-11-slim'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }

            when {
                changeset '**/worker/**'
            }

            steps {

                echo 'MONOPIPE: Running Unit Tets on worker app.'

                dir(path: 'worker') {
                    sh 'mvn clean test'
                }
            }
        }

        stage('monopipe-worker-package') {

            agent {
                docker {
                    image 'maven:3.8.5-jdk-11-slim'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }

            when {
                branch 'master'
                changeset '**/worker/**'
            }

            steps {

                echo 'MONOPIPE: Packaging worker app'

                dir(path: 'worker') {
                    sh 'mvn package -DskipTests'
                    archiveArtifacts(artifacts: '**/target/*.jar', fingerprint: true)
                }
            }
        }

        stage('monopipe-worker-docker-package') {

            agent any

            when {
                changeset '**/worker/**'
                branch 'master'
            }

            steps {

                echo 'MONOPIPE: Packaging worker app with docker'

                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                        def workerImage = docker.build("phigor/worker:v${env.BUILD_ID}", './worker')
                        workerImage.push()
                        workerImage.push("${env.BRANCH_NAME}")
                        workerImage.push('latest')
                    }
                }
            }
        }

        stage('monopipe-result-build') {

            agent {
                docker {
                    image 'node:8.16.0-alpine'
                }
            }

            when {
                changeset '**/result/**'
            }

            steps {

                echo 'MONOPIPE: Compiling result app..'

                dir(path: 'result') {
                    sh 'npm install'
                }
            }
        }

        stage('monopipe-result-test') {

            agent {
                docker {
                    image 'node:8.16.0-alpine'
                }
            }

            when {
                changeset '**/result/**'
            }

            steps {

                echo 'MONOPIPE: Running Unit Tests on result app..'

                dir(path: 'result') {
                    sh 'npm install'
                    sh 'npm test'
                }
            }
        }

        stage('monopipe-result-docker-package') {

            agent any

            when {
                changeset '**/result/**'
                branch 'master'
            }

            steps {

                echo 'MONOPIPE: Packaging result app with docker'

                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                        def resultImage = docker.build("phigor/result:v${env.BUILD_ID}", './result')
                        resultImage.push()
                        resultImage.push("${env.BRANCH_NAME}")
                        resultImage.push('latest')
                    }
                }
            }
        }

        stage('monopipe-vote-build') {

            agent {
                docker {
                    image 'python:2.7.16-slim'
                    args '--user root'
                }
            }

            when {
                changeset '**/vote/**'
            }

            steps {

                echo 'MONOPIPE: Compiling vote app.'

                dir(path: 'vote') {
                    sh 'pip install -r requirements.txt'
                }
            }
        }

        stage('monopipe-vote-test') {

            agent {
                docker {
                    image 'python:2.7.16-slim'
                    args '--user root'
                }

            }

            when {
                changeset '**/vote/**'
            }

            steps {

                echo 'MONOPIPE: Running Unit Tests on vote app.'

                dir(path: 'vote') {
                    sh 'pip install -r requirements.txt'
                    sh 'nosetests -v'
                }
            }
        }

        stage('monopipe-vote-integration'){

            agent any

            when{
                changeset "**/vote/**"
                branch 'master'
            }

            steps{

                echo 'MONOPIPE: Running Integration Tests on vote app'
                dir('vote'){
                    sh 'sh integration_test.sh'
                }
            }
        }

        stage('monopipe-vote-docker-package') {

            agent any

            steps {

                echo 'MONOPIPE: Packaging vote app with docker'

                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                        def voteImage = docker.build("phigor/vote:v${env.BUILD_ID}", "./vote")
                        voteImage.push()
                        voteImage.push("${env.BRANCH_NAME}")
                        voteImage.push("latest")
                    }
                }
            }
        }

        stage('monopipe-deploy-to-dev') {

            agent any

            when {
                branch 'master'
            }

            steps {
                echo 'MONOPIPE: Deploy instavote app with docker compose'
                sh 'docker-compose up -d'
            }
        }
    }

    post {
        always {
            echo 'Building mono pipeline for voting app is completed.'
        }
    }
}