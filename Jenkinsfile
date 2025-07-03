pipeline {
    agent any

    tools {
        maven 'maven3.9.1'
    }
    
    parameters {
        choice(name: 'DEPLOY_ENV', choices: ['blue', 'green'], description: 'Choose which environment to deploy: Blue or Green')
        choice(name: 'DOCKER_TAG', choices: ['blue', 'green'], description: 'Choose the Docker image tag for the deployment')
        booleanParam(name: 'SWITCH_TRAFFIC', defaultValue: false, description: 'Switch traffic between Blue and Green')
    }
    
    environment {
        IMAGE_NAME = "arvindpdige/bankapp"
        TAG = "${params.DOCKER_TAG}"  // The image tag now comes from the parameter
        KUBE_NAMESPACE = 'prod'
        // SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', credentialsId: 'git-token', url: 'https://github.com/arvindpdige/Blue-Green.git'
            }
        }

        
        // stage('SonarQube Analysis') {
        //     steps {
        //         withSonarQubeEnv('sonar') {
        //             sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=nodejsmysql -Dsonar.projectName=nodejsmysql"
        //         }
        //     }
        // }
        
        stage('Build') {
            steps {
                    bat 'mvn clean compile install package -Dmaven.test.failure.ignore=true'
            }
        }

        stage('Trivy FS Scan') {
            steps {
                bat "trivy fs --format table -o fs.html ."
            }
        }
        
        stage('Docker build') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerhub') {
                        bat "docker build -t ${IMAGE_NAME}:${TAG} ."
                    }
                }
            }
        }
        
        stage('Trivy Image Scan') {
            steps {
                bat "trivy image --format table -o image.html ${IMAGE_NAME}:${TAG}"
            }
        }
        
        stage('Docker Push Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerhub') {
                        bat "docker push ${IMAGE_NAME}:${TAG}"
                    }
                }
            }
        }
        
        stage('Deploy MySQL Deployment and Service') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'kube-config', variable: 'kube-config')]) {
                        bat "kubectl apply -f mysql-ds.yml -n ${KUBE_NAMESPACE}"  // Ensure you have the MySQL deployment YAML ready
                    }
                }
            }
        }
        
        stage('Deploy SVC-APP') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'kube-config', variable: 'kube-config')]) {
                        bat """ if ! kubectl get svc bankapp-service -n ${KUBE_NAMESPACE}; then
                                kubectl apply -f bankapp-service.yml -n ${KUBE_NAMESPACE}
                              fi
                        """
                   }
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def deploymentFile = ""
                    if (params.DEPLOY_ENV == 'blue') {
                        deploymentFile = 'app-deployment-blue.yml'
                    } else {
                        deploymentFile = 'app-deployment-green.yml'
                    }

                    withCredentials([file(credentialsId: 'kube-config', variable: 'kube-config')]) {
                        bat "kubectl apply -f ${deploymentFile} -n ${KUBE_NAMESPACE}"
                    }
                }
            }
        }
        
        stage('Switch Traffic Between Blue & Green Environment') {
            when {
                expression { return params.SWITCH_TRAFFIC }
            }
            steps {
                script {
                    def newEnv = params.DEPLOY_ENV

                    // Always switch traffic based on DEPLOY_ENV
                    withCredentials([file(credentialsId: 'kube-config', variable: 'kube-config')]) {
                        bat '''
                            kubectl patch service bankapp-service -p "{\\"spec\\": {\\"selector\\": {\\"app\\": \\"bankapp\\", \\"version\\": \\"''' + newEnv + '''\\"}}}" -n ${KUBE_NAMESPACE}
                        '''
                    }
                    echo "Traffic has been switched to the ${newEnv} environment."
                }
            }
        }
        
        stage('Verify Deployment') {
            steps {
                script {
                    def verifyEnv = params.DEPLOY_ENV
                    withCredentials([file(credentialsId: 'kube-config', variable: 'kube-config')]) {
                        bat """
                        kubectl get pods -l version=${verifyEnv} -n ${KUBE_NAMESPACE}
                        kubectl get svc bankapp-service -n ${KUBE_NAMESPACE}
                        """
                    }
                }
            }
        }
    }
}
