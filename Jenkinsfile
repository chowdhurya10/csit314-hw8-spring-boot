pipeline {
    agent any
    tools { maven '3.9.11' }
    options { timestamps(); skipStagesAfterUnstable() }
    environment {
        NEXUS_URL = 'http://hw8-nexus:8081/repository/hw8-releases'
        ARTIFACT = 'spring-boot-complete'
        VERSION = '1.0.0'
    }
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
        stage('Publish to Nexus') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'hw8-nexus-deploy',
                    usernameVariable: 'NEXUS_USER',
                    passwordVariable: 'NEXUS_PASS'
                )]) {
                    sh '''
                        set +x
                        umask 077
                        settings=$(mktemp)
                        trap 'rm -f "$settings"' EXIT
                        cat > "$settings" <<'SETTINGS'
<settings>
  <servers><server>
    <id>hw8-nexus</id>
    <username>${env.NEXUS_USER}</username>
    <password>${env.NEXUS_PASS}</password>
  </server></servers>
</settings>
SETTINGS
                        mvn -B -s "$settings" \
                          org.apache.maven.plugins:maven-deploy-plugin:3.2.0:deploy-file \
                          -DrepositoryId=hw8-nexus -Durl="$NEXUS_URL" \
                          -Dfile="target/$ARTIFACT-$VERSION.jar" -DpomFile=pom.xml
                        curl --fail --silent --show-error \
                          --user "$NEXUS_USER:$NEXUS_PASS" \
                          "$NEXUS_URL/com/example/$ARTIFACT/$VERSION/$ARTIFACT-$VERSION.jar" \
                          -o target/downloaded-from-nexus.jar
                        sha256sum "target/$ARTIFACT-$VERSION.jar" \
                          target/downloaded-from-nexus.jar
                        cmp "target/$ARTIFACT-$VERSION.jar" target/downloaded-from-nexus.jar
                    '''
                }
            }
        }
    }
    post {
        always { junit 'target/surefire-reports/*.xml' }
        success { echo 'Build and Nexus publication successful!' }
        failure { echo 'Build failed; inspect the console output.' }
    }
}
