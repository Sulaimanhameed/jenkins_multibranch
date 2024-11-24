
@Library('jenkins-shared-library@master') _
pipeline {
    agent {
        label 'docker-build-agent'
        }

    stages {
        stage('Git Checkout') {
            steps {
                gitCheckout(
                    branch: "main",
                    url: "https://github.com/spring-projects/spring-boot.git"
                )
            }
        }
        stage('Lint Dockerfile') {
            steps {
                container('hadolint') {
                    hadoLint()
                }
            }
        }
        stage('Build with Maven') {
            steps {
                container('maven') {
                    script {
                        maven.build()
                    }
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                container('kaniko') {
                    script {
                        kaniko.build()
                    }
                }
            }
        }
        stage('Trivy Scan') {
            steps {
                    container('trivy') {
                        script {
                            trivyScan.kaniko()
                        }
                    }
                }
        }
        stage('Push Docker Image') {
            when {
                allOf {
                    branch 'main'
                    not {
                        changeRequest()
                    }
                }
            }
            steps {
                container('kaniko') {
                    script {
                        kaniko.push('myubuntu890/jenkins-java-app', "1.0.${BUILD_NUMBER}", 'docker-credentials')
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                trivyNotification('trivy-report.html', 'sulaiman@crunchops.com')
            }
        }
    }
}
