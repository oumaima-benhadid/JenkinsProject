pipeline {
    agent any

    tools {
        maven '$M2_HOME'
        jdk '$JAVA_HOME'
    }

    environment {
        SONAR_INSTALL = 'sonarQube' 
        DOCKERHUB_CREDENTIALS = 'dockerhub-creds' // ID des credentials Docker Hub dans Jenkins
        IMAGE_NAME = 'omaimaabhd1807/springapp'
        IMAGE_TAG = 'latest'
        KUBECONFIG = '/var/lib/jenkins/.kube/config' 
    }

    stages {

        // ---------------- CI ----------------
        stage('1 - Checkout (Git)') {
            steps {
                checkout scm
            }
        }

        stage('2 - Maven Clean') {
            steps {
                echo 'Nettoyage du projet...'
                sh 'mvn -B clean'
            }
        }

        stage('3 - Maven Compile') {
            steps {
                echo 'Compilation du projet'
                sh 'mvn -B -DskipTests=true compile'
            }
        }

       

        stage('5 - Build & Archive JAR') {
            steps {
                echo 'Construction du package final'
                sh 'mvn -B -DskipTests=true package'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }

       

        // ---------------- CD ----------------
        stage('6 - Build Docker Image') {
            steps {
                script {
                    echo "Création de l'image Docker..."
                    sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                }
            }
        }

        stage('7 - Push Docker Image') {
        steps {
        script {
            withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                sh '''
                    echo "Connexion à Docker Hub..."
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push omaimaabhd1807/springapp:latest
                '''
            }
        }
    }
}


        stage('8 - Deploy to Minikube') {
    steps {
        echo "Déploiement MySQL et backend sur Minikube..."
        sh 'kubectl --kubeconfig=/var/lib/jenkins/.kube/config apply -f mysql-secret.yaml'
        sh 'kubectl --kubeconfig=/var/lib/jenkins/.kube/config apply -f mysql-deployment.yaml'
        sh 'kubectl --kubeconfig=/var/lib/jenkins/.kube/config apply -f restaurant-app-deployment.yaml'
        sh 'kubectl --kubeconfig=/var/lib/jenkins/.kube/config apply -f restaurant-app-service.yaml'
    }
}

    }

    post {
        success {
            echo 'Pipeline CI/CD terminé avec succès '
        }
        failure {
            echo 'Échec du pipeline '
        }
    }
}
