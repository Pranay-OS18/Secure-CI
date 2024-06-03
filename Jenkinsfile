
pipeline {
    agent any

    tools {
        jdk 'Open-JDK-17'
        maven 'Maven'
    }

    environment {
        SCANNER_HOME = tool 'Sonar-Scanner'
    }
    stages {

        stage('Git Checkout'){
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/Pranay-OS18/Secure-CI.git'
            }
        }

        stage('SAST Scan') {
            steps {
                withSonarQubeEnv('Sonar-Server') {
                        sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.organization=secure-ci  -Dsonar.projectName=secure-ci -Dsonar.projectKey=secure-ci_java-app-project \
                        -Dsonar.java.binaries=. '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialId: 'Sonar-Token'
                }

            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('SCA Scan') {
            steps {
                snykSecurity(
                    snykInstallation: 'Snyk', snykTokenId: 'Snyk-API-Token', failOnIssues: false, monitorProjectOnBuild: true, additionalArguments: '--sca --debug'
                )
            }
        }

        stage('Build Docker Image') {
            steps {
                withDockerRegistry(credentialsId: 'Docker-Cred', url: '', toolName: 'Docker') {
                    sh 'docker build -t pranay18cr/bg-app-image:v1'
                }
            }
        }
    }
}
