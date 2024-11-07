pipeline {
    agent any
    tools{
        nodejs 'npm23'
        maven 'mvn3'
    }
    environment{
       SCANNER_HOME = tool 'sonar-scanner'
       DOCKER_USER = 'mshow1980'
       APP_NAME = 'tuesday_app'
       IMAGE_NAME = "${DOCKER_USER}"+"/"+"${APP_NAME}"
       APP_VERSION = '1.5'
       IMAGE_TAG = "${APP_VERSION}-${BUILD_NUMER}"
       REGISTRY_CRED = 'docker-login'
    }
    stages {
        stage('CleanWS'){
            steps{
                script{
                    cleanWs()
                }
            }
        }
        stage('Checkout SCM'){
            steps{
                script{
                    checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/mshow1980/TUESDAY_APP.git']])
                }
            }
        }
        stage('SonarQube_Analysis'){
            steps{
                script{
                    withSonarQubeEnv('SonarQube') {
                        sh """
                        $SCANNER_HOME/bin/sonar-scanner\
                          -Dsonar.projectKey=tuesday-app \
                        -Dsonar.sources=. \
                        """
                    }
                }
            }
        }
        stage('Quality-gate_Analysis'){
            steps{
                script{
                    waitForQualityGate abortPipeline: false, credentialsId: 'SOnar-Token'
                }
            }
        }
        stage('OWASP-Dependency-Check'){
            steps{
                script{
                    dependencyCheck additionalArguments: '''-o "./"
                        -s "./"
                        -f "ALL"
                        -prettyPrint''', odcInstallation: 'OWASP_DC'
                    dependencyCheckPublisher pattern: 'Dependency-Check-Report.xml'
                }
            }
        }
        stage('File-System-Scan'){
            steps{
                script{
                   sh "trivy fs --format table -o trivyfs.html ."
                }
            }
        }
        stage('InstallingDependency'){
            steps{
                script{
                    sh ' npm install'
                }
            }
        }
        stage('BUILDING-IMAGE'){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker-login') {
                        docker_image = docker.build"${IMAGE_NAME}"
                    }
                }
            }
        }
        stage('Image-Scan'){
            steps{
                script{
                    sh 'trivy image --scanners vuln --scanners misconfig "${IMAGE_NAME}" --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table -o trvy-image.html '
                }
            }
        }
        stage('Image-Tag'){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker-login') {
                        docker_image.push("${IMAGE_NAME}")
                        docker_image.push('latet')
                    }
                }
            }
        }    
    }
}