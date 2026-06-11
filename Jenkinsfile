pipeline {
    agent any
    environment {
        SCANNER_HOME = tool 'mysonar'
    }
    stages {
        stage('code') {
            steps {
                git branch: 'main', url: 'https://github.com/Raghu-web/3-Tier-Full-Stack-yelp-camp.git'
            }
        }
        stage('CQA') {
            steps {
               withSonarQubeEnv('mysonar') {
                    sh '''
                    ${SCANNER_HOME}/bin/sonar-scanner \
                    -Dsonar.projectKey=myproject \
                    -Dsonar.projectName=myproject
                    '''
                }
            }
        }
        stage('quanlity gates') {
            steps {
                waitForQualityGate abortPipeline: false, credentialsId: 'mysonar'
            }
        }
        stage('Dependency checker') {
            steps {
                dependencyCheck additionalArguments: '--scan . --disableYarnAudit --disableNodeAudit', nvdCredentialsId: 'owasp-cred', odcInstallation: 'dp-check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Image') {
            steps {
                sh 'docker build -t appimage .'
            }
        }
        stage('Image scan') {
            steps {
                sh 'trivy image appimage'
            }
        }
        stage('Docker registry') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh 'docker tag appimage raghuram0024/eventapp:appimage'
                        sh 'docker push raghuram0024/eventapp:appimage'
                    }
                }
            }
        }
        stage('deploy') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'EKS_CLOUD', contextName: '', credentialsId: 'eks-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://A3085BD0242ED282921607449AD1C33D.gr7.us-east-1.eks.amazonaws.com') {
                    sh 'kubectl apply -f Manifests'
                }
            }
        }
    }
}
