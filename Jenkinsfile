pipeline {
    agent any

    environment {
        IMAGE_TAG = "release-0.0.${env.BUILD_NUMBER}"
        DOCKERHUB_CREDENTIALS = credentials('5f8b634a-148a-4067-b996-07b4b3276fba')
        DOCKERHUB_USERNAME = "idrisniyi94"
        DEPLOYMENT_NAME = "netflix-app"
        IMAGE_NAME = "${DOCKERHUB_USERNAME}/${DEPLOYMENT_NAME}:${IMAGE_TAG}"
        SCANNER_HOME = tool 'sonar-scanner'
        THE_MOVIE_DB_APIKEY = credentials('e4bc6a92-eba1-448c-ae39-bdae68277560')
        BRANCH_NAME = "${GIT_BRANCH.split('/)[1]}"
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Git Checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], userRemoteConfigs: [[url: 'https://github.com/stwins60/DevSecOps-Project-Netflixapp.git']]])
            }
        }
        stage('Sonarqube Analysis') {
            steps {
                script {
                    withSonarQubeEnv('sonar-server') {
                        sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=netflix-app -Dsonar.projectName=netflix-app"
                    }
                }
            }
        }
        stage('Quality Gate') {
            steps {
                script {
                    withSonarQubeEnv('sonar-server') {
                        waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                    }
                }
            }
        }
        stage('OWASP Vulnerability Scanner') {
            steps {
                script {
                    dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit --nvdApiKey f9c669af-b15f-4487-9d6d-d930c8f1b7a4', odcInstallation: 'DP-Check'
                    dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                }
            }
        }
        stage('Docker Login') {
            steps {
                script {
                    sh "echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin"
                }
            }
        }
        stage('Docker Build') {
            steps {
                script {
                    sh "docker build -t $IMAGE_NAME --build-arg TMDB_V3_API_KEY=$THE_MOVIE_DB_APIKEY ."
                }
            }
        }
        stage('Trivy Image Scan') {
            steps {
                script {
                    sh "trivy image $IMAGE_NAME"
                }
            }
        }
        stage('Docker Push') {
            steps {
                script {
                    sh "docker push $IMAGE_NAME"
                }
            }
        }
    }
    post {
        success {
            slackSend channel: '#alerts', color: 'good', message: "${currentBuild.currentResult}: \nJOB_NAME: ${env.JOB_NAME} \nBUILD_NUMBER: ${env.BUILD_NUMBER} \nBRANCH_NAME: ${env.BRANCH_NAME}. \n More Info ${env.BUILD_URL}"
        }
        failure {
            slackSend channel: '#alerts', color: 'danger', message: "${currentBuild.currentResult}: \nJOB_NAME: ${env.JOB_NAME} \nBUILD_NUMBER: ${env.BUILD_NUMBER} \nBRANCH_NAME: ${env.BRANCH_NAME}. \n More Info ${env.BUILD_URL}"
        }
    }
}