node {
    stage('Prep') {
        checkout scm
    }

    stage('Build') {
        def gradleContainer = docker.image('gradle:8-jdk8')
        gradleContainer.pull()
        
        gradleContainer.inside("-v /var/jenkins_home/.gradle:/home/gradle/.gradle") {
            sh 'cd complete && gradle build --no-daemon'
        }
    }

    stage('SonarQube Analysis') {
        def scannerHome = tool 'SonarScanner'
        
        withCredentials([string(credentialsId: 'sonarqube-token', variable: 'sonarLogin')]) {
            sh "${scannerHome}/bin/sonar-scanner " +
               "-Dsonar.host.url=http://sonarqube:9000 " +
               "-Dsonar.token='${sonarLogin}' " +
               "-Dsonar.projectName=gs-gradle " +
               "-Dsonar.projectVersion=${BUILD_NUMBER} " +
               "-Dsonar.projectKey=GS " +
               "-Dsonar.sources=complete/src/main/ " +
               "-Dsonar.tests=complete/src/test/ " +
               "-Dsonar.language=java " +
               "-Dsonar.java.binaries=."
        }
    }
}
