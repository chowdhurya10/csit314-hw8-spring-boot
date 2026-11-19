pipeline {
    agent any
    tools { maven '3.9.11' }
    options { timestamps(); skipStagesAfterUnstable() }
    stages {
        stage('Checkout Source Code') {
            steps {
                checkout scm
                sh """
                    cp -a complete/. .
                    sed -i 's#<version>0.0.1-SNAPSHOT</version>#<version>1.0.0</version>#' pom.xml
                """
            }
        }
        stage('Test') {
            steps { sh 'mvn -B clean test' }
        }
        stage('Build and Package') {
            steps { sh 'mvn -B package -DskipTests' }
        }
        stage('Archive Artifacts') {
            steps { archiveArtifacts artifacts: 'target/*.jar', fingerprint: true }
        }
    }
    post {
        always { junit 'target/surefire-reports/*.xml' }
        success { echo 'Build successful!' }
        failure { echo 'Build failed!' }
    }
}
