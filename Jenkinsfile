pipeline {
  agent any
  tools {
    maven 'M2_HOME'     
    jdk 'JAVA_HOME'        
  }
  environment {
    SONAR_INSTALL = 'sonarQube' 
  }
  stages {
    stage('1 - Checkout (git)') {
      steps {
        checkout scm
      }
    }

    stage('2 - Maven clean') {
      steps {
        sh 'mvn -B clean'
      }
    }

    stage('3 - Maven compile (package)') {
      steps {
        sh 'mvn -B -DskipTests=true package'
      }
    }

    stage('4 - SonarQube analysis') {
      steps {
        withSonarQubeEnv("${SONAR_INSTALL}") {
          sh 'mvn -B sonar:sonar'
        }
      }
    }

    stage('5 - Build final & archive jar') {
      steps {
        sh 'mvn -B -DskipTests=false package'
        archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
      }
    }

    stage('Quality Gate') {
      steps {
        timeout(time: 15, unit: 'MINUTES') {
          waitForQualityGate abortPipeline: true
        }
      }
    }
  }

  post {
    success {
      echo 'Pipeline terminé: SUCCESS'
    }
    failure {
      echo 'Pipeline terminé: FAILURE'
    }
  }
}
