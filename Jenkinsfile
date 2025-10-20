pipeline {
  agent any

  tools {
    // Remplace par les noms exacts définis dans Jenkins
    jdk '$JAVA_HOME'
    maven '$M2_HOME'
  }

  environment {
    SONAR_INSTALL = 'sonarQube'
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

    stage('4 - SonarQube Analysis') {
      steps {
        echo 'Lancement de l’analyse SonarQube...'
        withEnv([
            "JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64",
            "PATH=/usr/lib/jvm/java-17-openjdk-amd64/bin:${env.PATH}",
            "M2_HOME=/usr/share/maven",
            "PATH=/usr/share/maven/bin:${env.PATH}"
        ]) {
            withSonarQubeEnv('sonarQube') {
                sh 'mvn -B sonar:sonar'
            }
        }
      }
    }

    stage('5 - Build & Archive JAR') {
      steps {
        echo 'Construction du package final...'
        sh 'mvn -B -DskipTests=true package'
        archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
      }
    }

    stage('Quality Gate') {
      steps {
        echo 'Vérification du Quality Gate SonarQube...'
        timeout(time: 15, unit: 'MINUTES') {
          waitForQualityGate abortPipeline: true
        }
      }
    }

    stage('6 - Docker Build & Push') {
      steps {
        echo 'Construction et push de l’image Docker...'
        sh """
          docker login -u omaimaabhd1807 -p \$DOCKERHUB_PASSWORD
          docker build -t ${DOCKER_IMAGE} .
          docker push ${DOCKER_IMAGE}
        """
      }
    }

    stage('7 - Deploy MySQL') {
      steps {
        echo 'Déploiement de MySQL sur Kubernetes...'
        sh "kubectl apply -f k8s/mysql-deployment.yaml -n ${NAMESPACE}"
      }
    }

    stage('8 - Deploy Spring Boot') {
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
