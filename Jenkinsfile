pipeline {
    agent any

    tools {
        maven "mavenpractice"
        jdk   "JDK17"
    }

    parameters {
        string(name: 'AWS_ACCOUNT_ID', defaultValue: '', description: 'AWS account that owns the ECR registry')
        string(name: 'AWS_REGION',     defaultValue: 'us-east-1', description: 'Region of the ECR registry')
    }

    environment {
        registryCredential = "ecr:${params.AWS_REGION}:awscreds"
        ecrRegistry        = "${params.AWS_ACCOUNT_ID}.dkr.ecr.${params.AWS_REGION}.amazonaws.com"
        imageName          = "${ecrRegistry}/appimg"
    }

    options {
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    stages {

        stage('Validate parameters') {
            steps {
                script {
                    if (!params.AWS_ACCOUNT_ID?.trim()) {
                        error 'AWS_ACCOUNT_ID is required'
                    }
                }
            }
        }

        stage('Fetch code') {
            steps {
                git branch: 'docker', url: 'https://github.com/hkhcoder/vprofile-project.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn install -DskipTests'
            }
            post {
                success {
                    echo 'Archiving artifact...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

        stage('Unit tests') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Checkstyle analysis') {
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
        }

        stage('SonarQube analysis') {
            environment {
                scannerHome = tool 'sonar6.2'
            }
            steps {
                withSonarQubeEnv('sonarserver') {
                    sh '''${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=vprofile \
                        -Dsonar.projectName=vprofile \
                        -Dsonar.projectVersion=1.0 \
                        -Dsonar.sources=src/ \
                        -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                        -Dsonar.junit.reportsPath=target/surefire-reports/ \
                        -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                        -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build app image') {
            steps {
                script {
                    dockerImage = docker.build("${imageName}:${BUILD_NUMBER}", "./Docker-files/app/multistage/")
                }
            }
        }

        stage('Push image to ECR') {
            steps {
                script {
                    docker.withRegistry("https://${ecrRegistry}", registryCredential) {
                        dockerImage.push("${BUILD_NUMBER}")
                        dockerImage.push('latest')
                    }
                }
            }
        }
    }

    post {
        always {
            // Remove only the images built by this job, not every image on the agent
            sh "docker rmi -f ${imageName}:${BUILD_NUMBER} ${imageName}:latest || true"
        }
    }
}
