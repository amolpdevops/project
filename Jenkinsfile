pipeline {

    agent {
        node {
            customWorkspace '/mnt/my-project'
        }
    }

    environment {
        DOCKER = credentials('docker_creds')
    }

    stages {

		stage('Clean Workspace') {
			steps {
				sh '''
				echo "===== Cleaning Workspace ====="

				rm -rf *
				'''
			}
		}

        stage('Checkout Source Code') {
            steps {
                sh '''
                echo "========== Checkout Source Code =========="

                git clone -b $GIT_BRANCH $GIT_URL .

                echo "Repository Cloned Successfully"

                pwd
                ls -ltr
                '''
            }
        }

        stage('Verify Installed Tools') {
            steps {
                sh '''
                echo "========== Java Version =========="
                java -version

                echo "========== Maven Version =========="
                mvn -version

                echo "========== Git Version =========="
                git --version

                echo "========== Docker Version =========="
                docker --version

                echo "========== AWS CLI Version =========="
                aws --version
                '''
            }
        }

        stage('Build Application') {
            steps {
                sh '''
                echo "========== Building Java Application =========="

                mvn clean install

                echo "Build Completed Successfully"
                '''
            }
        }

        stage('Verify WAR File') {
            steps {
                sh '''
                echo "========== Generated WAR File =========="

                ls -lh target/

                find target -name "*.war"
                '''
            }
        }

        stage('Upload WAR to S3') {
            steps {
                sh '''
                echo "========== Uploading WAR to S3 =========="

                aws s3 cp target/*.war s3://test-bucket-15956/

                echo "WAR Uploaded Successfully"
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                echo "========== Building Docker Image =========="

                docker build -t $DOCKER_IMAGE:$IMAGE_TAG .

                docker images
                '''
            }
        }

        stage('Docker Hub Login') {
            steps {
                sh '''
                echo "========== Docker Hub Login =========="

                echo "$DOCKER_PSW" | docker login \
                -u "$DOCKER_USR" \
                --password-stdin
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                sh '''
                echo "========== Pushing Docker Image =========="

                docker push $DOCKER_IMAGE:$IMAGE_TAG

                echo "Docker Image Pushed Successfully"
                '''
            }
        }

    }

    post {

        success {
            echo "=========================================="
            echo "CI Pipeline Completed Successfully"
            echo "=========================================="
        }

        failure {
            echo "=========================================="
            echo "CI Pipeline Failed"
            echo "=========================================="
        }

        always {
            sh 'docker logout || true'
        }
    }
}
