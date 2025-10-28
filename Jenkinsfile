pipeline {
    agent any
    tools {
        nodejs 'node17'
        jdk 'jdk17'
    }
    environment{
        SCANNER_HOME=tool 'sonarqube'
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Git Checkout') {
            steps {
                git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/Namrathaaaaaa/DevSecOps-Starbucks.git'
            }
        }
        stage('Sonarqube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh '''
                    $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=starbucks \
                    -Dsonar.projectKey=starbucks
                    '''
                }
            }
        }
        stage('Installing Dependencies') {
            steps {
                sh 'npm install'
            }
        }
        stage('Trivy Scan') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage('Docker Image Build') {
            steps {
                withDockerRegistry(credentialsId: 'docker-cred') {
                    sh 'docker build -t namratha3/starbucks:v2 .'
                    sh 'docker tag namratha3/starbucks:v2 namratha3/starbucks:latest'
                    sh 'docker push namratha3/starbucks:latest'
                }
            }
        }
        stage('Trivy Check') {
            steps {
                sh "trivy image namratha3/starbucks:latest > trivyimage.txt" 
            }
        }
        stage('Docker deploy') {
            steps {
                sh "docker run --name starbucks -dp 3000:3000 namratha3/starbucks:latest"
            }
        }
    }
}
post {
    always {
        script {
            def buildStatus = currentBuild.currentResult
            def buildUser = currentBuild.getBuildCauses('hudson.model.Cause$UserIdCause')[0]?.userId ?: 'Github User'
            
            emailext (
                subject: "Pipeline ${buildStatus}: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                    <p>This is a Jenkins starbucks CICD pipeline status.</p>
                    <p>Project: ${env.JOB_NAME}</p>
                    <p>Build Number: ${env.BUILD_NUMBER}</p>
                    <p>Build Status: ${buildStatus}</p>
                    <p>Started by: ${buildUser}</p>
                    <p>Build URL: <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                """,
                to: 'mohdaseemakram19@gmail.com',
                from: 'mohdaseemakram19@gmail.com',
                replyTo: 'mohdaseemakram19@gmail.com',
                mimeType: 'text/html',
                attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
            )
           }
       }

    }

}
