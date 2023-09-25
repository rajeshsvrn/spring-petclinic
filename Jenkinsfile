node {
    // Define your GitHub repository URL and credentials
    def githubRepoUrl = 'https://github.com/rajeshsvrn/spring-petclinic.git'
    def credentialsId = 'your-credentials-id' // You should replace this with your actual credential ID

    // Checkout the GitHub repository
    stage('Checkout') {
        checkout([$class: 'GitSCM',
            branches: [[name: 'main']], // Replace 'master' with your branch
            doGenerateSubmoduleConfigurations: false,
            extensions: [[$class: 'CloneOption', depth: 1, noTags: true, reference: '', shallow: true]],
            userRemoteConfigs: [[credentialsId: credentialsId, url: githubRepoUrl]]
        ])
    }

    // Add more stages for your build, test, and deployment steps here

     // Build the Maven application
    stage('Build') {
            // Set up the Maven environment (assuming you have Maven installed on your Jenkins agent)
            def mvnHome = tool name: 'Maven', type: 'hudson.tasks.Maven$MavenInstallation'
            env.PATH = "${mvnHome}/bin:${env.PATH}"

            // Execute the Maven build
            sh "mvn clean package" // Adjust the Maven goals as needed
        }
    

          // Define a post-build step for the "Build" stage to run JUnit tests
    post('Junit Test')  {
            // Run JUnit tests as a post-build step for the "Build" stage
            step([$class: 'JUnitResultArchiver', testResults: '**/target/surefire-reports/*.xml'])
        }
    }


}
