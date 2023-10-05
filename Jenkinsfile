node {
    // Define your GitHub repository URL and credentials
    def githubRepoUrl = 'https://github.com/rajeshsvrn/spring-petclinic.git'
    def credentialsId = 'your-credentials-id' // You should replace this with your actual credential ID

    def NEXUS_VERSION = "nexus3"
    def NEXUS_PROTOCOL = "http"
    def NEXUS_URL = "20.232.199.54:8081"
    def NEXUS_REPOSITORY = "maven-hosted"
    def NEXUS_CREDENTIAL_ID = "nexus"

    // Checkout the GitHub repository
    stage('Checkout') {
        checkout([$class: 'GitSCM',
            branches: [[name: 'main']], // Replace 'master' with your branch
            doGenerateSubmoduleConfigurations: false,
            extensions: [[$class: 'CloneOption', depth: 1, noTags: true, reference: '', shallow: true]],
            userRemoteConfigs: [[credentialsId: credentialsId, url: githubRepoUrl]]
        ])
    }
try {
    // Add more stages for your build, test, and deployment steps here

     // Build the Maven application
    stage('Build SW') {
            // Set up the Maven environment (assuming you have Maven installed on your Jenkins agent)
            def mvnHome = tool name: 'Maven', type: 'hudson.tasks.Maven$MavenInstallation'
            env.PATH = "${mvnHome}/bin:${env.PATH}"

            // Execute the Maven build
            sh "mvn clean package" // Adjust the Maven goals as needed
        }
} catch (Exception e) {
        currentBuild.result = 'FAILURE'
        error("Exception caught: ${e.message}") e  // Re-throw the exception to mark the build as a failure
    } finally {
      // Post-build stage
   stage('Junit Test') {
            if (currentBuild.resultIsBetterOrEqualTo('SUCCESS')) {
                // Run JUnit tests only if the "Build" stage is successful
                junit '**/target/surefire-reports/*.xml' // Path to your JUnit test report XML files
            }
        }
    }

 stage('SonarQube Analysis') {
            def scannerHome = tool 'sonarqube' // Ensure you have configured the "SonarQube Scanner" tool in Jenkins

            withSonarQubeEnv(credentialsId: 'sonarqube', installationName: 'sonarqube') {
                sh """
                    ${scannerHome}/bin/sonar-scanner \
                    -Dsonar.projectKey=petclinic \
                    -Dsonar.projectName=petclinic \
                    -Dsonar.projectVersion=1.0 \
                    -Dsonar.sources=src/main \
                    -Dsonar.tests=src/test \
                    -Dsonar.java.binaries=target/classes \
                    -Dsonar.language=java \
                    -Dsonar.sourceEncoding=UTF-8 \
                    -Dsonar.java.libraries=target/classes
                """
            }
        }
 stage("Quality Gate check"){
          timeout(time: 1, unit: 'HOURS') {
              def qg = waitForQualityGate()
              if (qg.status != 'OK') {
                  error "Pipeline aborted due to quality gate failure: ${qg.status}"
              }
          }
      }

stage("Publish artifact to nexus") {
    pom = readMavenPom file: "pom.xml";
    filesByGlob = findFiles(glob: "target/*.${pom.packaging}"); 
    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
    artifactPath = filesByGlob[0].path;
    artifactExists = fileExists artifactPath;
    if(artifactExists) {
        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";
        nexusArtifactUploader(
            nexusVersion: NEXUS_VERSION,
            protocol: NEXUS_PROTOCOL,
            nexusUrl: NEXUS_URL,
            groupId: pom.groupId,
            version: pom.version,
            repository: NEXUS_REPOSITORY,
            credentialsId: NEXUS_CREDENTIAL_ID,
            artifacts: [
                [artifactId: pom.artifactId,
                classifier: '',
                file: artifactPath,
                type: pom.packaging],

                [artifactId: pom.artifactId,
                classifier: '',
                file: "pom.xml",
                type: "pom"]
            ]
        );

    } else {
        error "*** File: ${artifactPath}, could not be found";
    }
}

node {
    def ACR_NAME = 'petcliniccontainer'
    def IMAGE_NAME = 'petimage'
    def IMAGE_TAG = 'petimage'
    //def AZURE_CREDENTIALS_ID = '25559edf-b103-462c-86d7-eb4259902a5d'


    stage('Build and Push Container Image') {
        try {
            // Authenticate to Azure using Azure Service Principal credentials
            withCredentials([azureServicePrincipal(credentialsId: 'AZURE_CREDENTIALS_ID', 
                                                    subscriptionId: '820b6969-ff53-431e-89cc-0377b9dcbab2',
                                                    resourceGroup: 'CICD-gr')]) {

                //echo current directory
                sh "pwd"
                
                
                // Build the Docker image
                sh "docker build -f /var/lib/jenkins/workspace/CICD_project/Dockerfile -t $IMAGE_NAME:$IMAGE_TAG ."


                // Tag the Docker image for ACR
                sh "docker tag $IMAGE_NAME:$IMAGE_TAG $ACR_NAME.azurecr.io/$IMAGE_NAME:$IMAGE_TAG"

                // Push the Docker image to ACR
                sh "docker push $ACR_NAME.azurecr.io/$IMAGE_NAME:$IMAGE_TAG"

                
            }
        } catch (Exception e) {
            currentBuild.result = 'FAILURE'
            throw e
        }
    }
}

    
}   //node end






