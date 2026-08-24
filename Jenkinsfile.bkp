pipeline {

    agent {
        node {
            label 'Agent-1'
        }
    }

    environment {

        AWS_REGION = 'us-east-1'

        AWS_ACCOUNT_ID = '453531893439'

        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"

        IMAGE_NAME = 'roboshop/cart'

        IMAGE_TAG = "${env.GIT_COMMIT.take(7)}"

        // SONARQUBE = 'sonarqube'

        GITOPS_REPO = 'https://github.com/aws-devopsprocloud/roboshop-gitops.git'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        // stage('SonarQube Scan') {
        //     steps {

        //         withSonarQubeEnv("${SONARQUBE}") {

        //             sh '''
        //                 sonar-scanner \
        //                   -Dsonar.projectKey=roboshop-cart \
        //                   -Dsonar.sources=cart
        //             '''
        //         }
        //     }
        // }

        // stage('Quality Gate') {
        //     steps {

        //         timeout(time: 5, unit: 'MINUTES') {

        //             waitForQualityGate abortPipeline: true
        //         }
        //     }
        // }

        // stage('Trivy Filesystem Scan') {
        //     steps {

        //         sh '''
        //             trivy fs \
        //               --severity HIGH,CRITICAL \
        //               --exit-code 1 \
        //               .
        //         '''
        //     }
        // }

        stage('Docker Build') {
            steps {

                sh '''
                    docker build \
                      -t ${ECR_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} \
                      ./cart
                '''
            }
        }

        // stage('Trivy Image Scan') {
        //     steps {

        //         sh '''
        //             trivy image \
        //               --severity HIGH,CRITICAL \
        //               --exit-code 1 \
        //               ${ECR_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
        //         '''
        //     }
        // }

        stage('Login to ECR') {
            steps {

                sh '''
                    aws ecr get-login-password \
                      --region ${AWS_REGION} |
                    docker login \
                      --username AWS \
                      --password-stdin \
                      ${ECR_REGISTRY}
                '''
            }
        }

        stage('Push Image') {
            steps {

                sh '''
                    docker push \
                      ${ECR_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }

    //     stage('Update GitOps') {
    //         steps {

    //             sh '''
    //                 rm -rf gitops

    //                 git clone \
    //                   ${GITOPS_REPO} \
    //                   gitops

    //                 cd gitops

    //                 sed -i.bak \
    //                   "s/imageVersion:.*/imageVersion: \\"${IMAGE_TAG}\\"/" \
    //                   charts/roboshop/values.yaml

    //                 rm -f charts/roboshop/values.yaml.bak

    //                 git config user.name "Jenkins"

    //                 git config user.email "jenkins@devopsprocloud.in"

    //                 git add .

    //                 git commit \
    //                   -m "Update cart image to ${IMAGE_TAG}"

    //                 git push origin main
    //             '''
    //         }
    //     }
    // }
        stage('Update GitOps') {

            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: 'github-pat',
                        usernameVariable: 'GIT_USERNAME',
                        passwordVariable: 'GIT_PASSWORD'
                    )
                ]) {

                    sh '''
                        rm -rf gitops

                        git clone \
                        https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/aws-devopsprocloud/roboshop-gitops.git \
                        gitops

                        cd gitops

                        sed -i.bak \
                        's/imageVersion:.*/imageVersion: "'${IMAGE_TAG}'"/' \
                        charts/roboshop/values.yaml

                        rm -f charts/roboshop/values.yaml.bak

                        git config user.name "Jenkins"

                        git config user.email "jenkins@devopsprocloud.in"

                        git add charts/roboshop/values.yaml

                        git commit \
                        -m "Update cart image to ${IMAGE_TAG}"

                        git push origin main
                    '''
                }
            }
        }
    }
    post {

        success {
            echo "CI completed successfully"
        }

        failure {
            echo "CI pipeline failed"
        }
    }
}