node {
    // 1. Fetch the absolute freshest code from your personal GitHub fork
    stage('Prep') {
        checkout scm
    }

    // 2. Compile the Java Application utilizing your global path fix
    stage('Build') {
        def gradleContainer = docker.image('gradle:8-jdk8')
        gradleContainer.pull()
        
        // We isolate the native services away from the locked host mounts
        gradleContainer.inside("-v /var/jenkins_home/.gradle:/home/gradle/.gradle") {
            sh 'cd complete && gradle build --no-daemon'
        }
    }

    // 3. Scan code metrics and securely stream to SonarQube port 9000
    stage('SonarQube Analysis') {
        // Automatically fetches the SonarScanner CLI binary from your Global Tool configuration
        def scannerHome = tool 'SonarScanner'
        
        // Safely masks your user token into the environment variable '$sonarLogin'
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
