pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "busapooja/instagram-ui"
        SONARQUBE_SERVER = "SonarQube"
     //   NEXUS_URL = "192.168.0.20:8082"
    }

    stages {

        stage('Git Checkout') {
    steps {
        checkout([
            $class: 'GitSCM',
            branches: [[name: '*/main']],
            userRemoteConfigs: [[
                url: 'https://github.com/PoojaBusa09/instagram-devops-project.git'
            ]]
        ])
    }
}

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_SERVER}") {
                    sh '''
                    sonar-scanner \
                    -Dsonar.projectKey=instagram-devops-project \
                    -Dsonar.projectName=instagram-devops-project \
                    -Dsonar.sources=.
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t $DOCKER_IMAGE:latest ."
            }
        }

        stage('DockerHub Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-hub-cred',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh 'echo $PASS | docker login -u $USER --password-stdin'
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                sh "docker push $DOCKER_IMAGE:latest"
            }
        }

        stage('Push to Nexus') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'nexus-cred',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh """
                    docker login $NEXUS_URL -u $USER -p $PASS
                    docker tag $DOCKER_IMAGE:latest $NEXUS_URL/$DOCKER_IMAGE:latest
                    docker push $NEXUS_URL/$DOCKER_IMAGE:latest
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                kubectl apply -f deployment.yaml
                kubectl apply -f service.yaml
                '''
            }
        }
    }
}
