pipeline {
  agent any

  tools {
    // Remplace par les noms exacts définis dans Jenkins
    jdk '$JAVA_HOME'
    maven '$M2_HOME'
  }

  environment {
    DOCKER_IMAGE = 'omaimaabhd1807/springapp:latest'
    NAMESPACE = 'devops'
  }

  stages {

    stage('1 - Checkout (Git)') {
      steps {
        echo 'Récupération du code source...'
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
        echo 'Compilation du projet...'
        sh 'mvn -B -DskipTests=true compile'
      }
    }

    stage('4 - Build & Archive JAR') {
      steps {
        echo 'Construction du package final...'
        sh 'mvn -B -DskipTests=true package'
        archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
      }
    }

    stage('5 - Docker Build & Push') {
      steps {
        echo 'Construction et push de l’image Docker...'
        sh """
          docker login -u omaimaabhd1807 -p \$DOCKERHUB_PASSWORD
          docker build -t ${DOCKER_IMAGE} .
          docker push ${DOCKER_IMAGE}
        """
      }
    }

    stage('6 - Deploy MySQL') {
      steps {
        echo 'Déploiement de MySQL sur Kubernetes...'
        sh "kubectl apply -f k8s/mysql_deployment.yaml -n ${NAMESPACE}"
      }
    }

    stage('7 - Deploy Spring Boot') {
      steps {
        echo 'Déploiement de l’application Spring Boot sur Kubernetes...'
        sh "kubectl apply -f k8s/spring-deployment.yaml -n ${NAMESPACE}"
      }
    }

  }

  post {
    success {
      echo 'Pipeline terminé avec succès.'
    }
    failure {
      echo 'Échec du pipeline.'
    }
  }
}
