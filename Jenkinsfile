pipeline {
  agent any

  environment {
    JAVA_HOME = '/usr/lib/jvm/java-17-openjdk-amd64'
    PATH = "${JAVA_HOME}/bin:${env.PATH}"
    M2_HOME = '/usr/share/maven'   // Remplace par le chemin réel vers Maven si différent
    PATH = "${M2_HOME}/bin:${env.PATH}"
    SONAR_INSTALL = 'sonarQube'
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
        withSonarQubeEnv("${SONAR_INSTALL}") {
          sh 'mvn -B sonar:sonar'
        }
      }
    }

    stage('5 - Build & Archive JAR') {
      steps {
        echo 'Construction du package final...'
        sh 'mvn -B -DskipTests=false package'
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
