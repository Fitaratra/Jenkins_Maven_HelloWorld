pipeline {
  agent any

  tools {
    jdk   'JDK17'  // défini dans Global Tool Configuration
    maven 'Maven3'     // idem
  }

  environment {
    SONAR_HOST = 'http://sonarqube:9000'  // reachable via réseau Docker
  }

  stages {
    stage('Checkout') {
      steps {
        // ajoute credentialsId: 'git-cred' si repo privé
        git branch: 'main', url: 'https://github.com/Fitaratra/Jenkins_Maven_HelloWorld.git'
      }
    }

    stage('Build') {
      steps {
        sh 'mvn -B -U clean package'
      }
    }

    stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv('SonarQubeServer') {
      withCredentials([string(credentialsId: 'sonar_token', variable: 'SONAR_TOKEN')]) {
        sh '''
          mvn clean verify sonar:sonar \
            -Dsonar.projectKey=jenkins_maven \
            -Dsonar.host.url=http://sonarqube:9000 \
            -Dsonar.login=${SONAR_TOKEN}
        '''
      }
    }
        
      }
      
    }

    stage('Deploy to Nexus') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'nexus', usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
          sh '''
            cat > settings.xml <<EOF
<settings>
  <servers>
    <server>
      <id>nexus-releases</id>
      <username>${NEXUS_USER}</username>
      <password>${NEXUS_PASS}</password>
    </server>
    <server>
      <id>nexus-snapshots</id>
      <username>${NEXUS_USER}</username>
      <password>${NEXUS_PASS}</password>
    </server>
  </servers>
</settings>
EOF
            mvn -B -s settings.xml -DskipTests deploy
          '''
        }
      }
    }
  }

  post {
    always {
      echo "Résultat : ${currentBuild.currentResult}"
    }
  }
}
