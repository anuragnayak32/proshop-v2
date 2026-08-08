// pipeline {
//     agent any

//     stages {

//         stage('Clone Verification') {
//             steps {
//                 sh 'pwd'
//                 sh 'ls -la'
//             }
//         }

//         stage ('Build Backend Image') {
//             steps {
//                 sh 'docker build -t proshop-backend:$BUILD_NUMBER -f Dockerfile.backend .'
//             }
//         }

//         stage ('Tag & push Backend Image') {
//             steps {
//                 sh 'docker tag proshop-backend:$BUILD_NUMBER anurag32/proshop-backend:$BUILD_NUMBER'
//                 sh 'docker push anurag32/proshop-backend:$BUILD_NUMBER'
//             }
//         }

//         stage('BUILD Frontend Image') {
//             steps {
//                 sh 'docker build -t proshop-frontend:$BUILD_NUMBER -f Dockerfile.frontend .'
//             }
//         }

//         stage('Tag & Push Frontend Image') {
//             steps {
//                 sh 'docker tag proshop-frontend:$BUILD_NUMBER anurag32/proshop-frontend:$BUILD_NUMBER'
//                 sh 'docker push anurag32/proshop-frontend:$BUILD_NUMBER'
//             }
//         }
//     }
// }




pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS = credentials('dockerhub')
        KUBECONFIG = '/root/.kube/jenkins-config'
    }

    stages {
        stage('Clone Verification') {
            steps {
                sh 'pwd'
                sh 'ls -la'
            }
        }

        stage('Docker Login'){
            steps {
                sh '''
                echo "$DOCKER_CREDENTIALS_PSW" | docker login -u "$DOCKER_CREDENTIALS_USR" --password-stdin
                '''
            }
        }

        stage ('Build Backend Image') {
            steps {
                sh 'docker build -t proshop-backend:$BUILD_NUMBER -f Dockerfile.backend .'
            }
        }

        stage('Tag & Push Backend Image') {
            steps {
                sh 'docker tag proshop-backend:$BUILD_NUMBER anurag32/proshop-backend:$BUILD_NUMBER'
                sh 'docker push anurag32/proshop-backend:$BUILD_NUMBER'
            }
        }

        stage('Build Frontend Image') {
            steps {
                sh 'docker build -t proshop-frontend:$BUILD_NUMBER -f Dockerfile.frontend .'
            }
        }

        stage('Tag & Push Frontend Image') {
            steps {
                sh 'docker tag proshop-frontend:$BUILD_NUMBER anurag32/proshop-frontend:$BUILD_NUMBER'
                sh 'docker push anurag32/proshop-frontend:$BUILD_NUMBER'
            }
        }


        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    helm upgrade --install proshop-dev ./helm/proshop-chart \
                    -f ./helm/proshop-chart/values-dev.yaml \
                    --set backend.image.repository=anurag32/proshop-backend \
                    --set backend.image.tag=$BUILD_NUMBER \
                    --set backend.image.pullPolicy=IfNotPresent \
                    --set frontend.image.repository=anurag32/proshop-frontend \
                    --set frontend.image.tag=$BUILD_NUMBER \
                    --set frontend.image.pullPolicy=IfNotPresent
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    kubectl get pods
                    kubectl get deployments
                    helm status proshop-dev
                '''
            }
        }

        stage('Docker Logout') {
            steps {
                sh 'docker logout'
            }
        }


       stage('Cleanup') {
            steps {
                sh '''
                    docker image prune -f
                    docker builder prune -f
                '''
            }
        }
    }
}