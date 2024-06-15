pipeline {
    agent any 
    tools {
        jdk 'jdk'
        nodejs 'nodejs'
    }
    environment  {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('Cleaning Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'master', url: 'https://github.com/DiNeSHhMj/End-to-End-Kubernetes-DevSecOps-Tetris-Project.git'
            }
        }
        stage('Sonarqube Analysis') {
            steps {
                dir('Tetris-V1') {
                    withSonarQubeEnv('sonar-server') {
                        sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=TetrisVersion1.0 \
                        -Dsonar.projectKey=TetrisVersion1.0 '''
                    }
                }
            }
        }
        stage('Quality Check') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token' 
                }
            }
        }
        stage('Installing Dependencies') {
            steps {
                dir('Tetris-V1') {
                    sh 'npm install'
                }
            }
        }
        // stage('OWASP Dependency-Check Scan') {
        //     steps {
        //         dir('Tetris-V1') {
        //             dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
        //             dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
        //         }
        //     }
        // }
        stage('Trivy File Scan') {
            steps {
                dir('Tetris-V1') {
                    sh 'trivy fs . > trivyfs.txt'
                }
            }
        }
        stage("Docker Image Build") {
            steps {
                script {
                    dir('Tetris-V1') {
                        withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {   
                            //sh 'docker system prune -f'
                            //sh 'docker container prune -f'
                            sh 'docker build -t tetris_game-v1 .'
                        }
                    }
                }
            }
        }
        stage("Docker Image Pushing") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {   
                        sh 'docker tag tetris_game-v1 dinesh808/tetris_game-v1:${BUILD_NUMBER}'
                        sh 'docker push dinesh808/tetris_game-v1:${BUILD_NUMBER}'
                    }
                }
            }
        }
        stage("TRIVY Image Scan") {
            steps {
                sh 'trivy image dinesh808/tetris_game-v1:${BUILD_NUMBER} > trivyimage.txt' 
            }
        }
        stage('Checkout Code') {
            steps {
                git branch: 'master', url: 'https://github.com/DiNeSHhMj/End-to-End-Kubernetes-DevSecOps-Tetris-Project.git'
            }
        }
        stage('Update Deployment file') {
            environment {
                GIT_REPO_NAME = "End-to-End-Kubernetes-DevSecOps-Tetris-Project"
                GIT_USER_NAME = "DiNeSHhMj"
            }
            steps {
                dir('Manifest-file') {
                    withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                        sh '''
                            git config user.email "dineshmajumdar18@gmail.com"
                            git config user.name "DiNeSHhMj"
                            BUILD_NUMBER=${BUILD_NUMBER}
                            echo $BUILD_NUMBER
                            imageTag=$(grep -oP '(?<=tetris_game-v1:)[^ ]+' deployment-service.yml)
                            echo $imageTag
                            sed -i "s/tetris_game-v1:${imageTag}/tetris_game-v1:${BUILD_NUMBER}/" deployment-service.yml
                            git add deployment-service.yml
                            git commit -m "Update deployment Image to version \${BUILD_NUMBER}"
                            git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:master
                        '''
                    }
                }
            }
        }
    }
}