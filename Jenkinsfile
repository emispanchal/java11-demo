pipeline {
    agent any
    environment {
        SONAR_TOKEN= '5dacf7a562852986fe3fc404b72a00cbb8404111'
        APP_HOME='/home/app'
        PRAGRA_BATCH='devs'
    }
    options { 
        quietPeriod(30) 
    }
    parameters { 
            choice(name: 'ENV_TO_DEPLOY', 
            choices: ['ST', 'UAT', 'STAGING'], description: 'Select a Env to deploy') 

             booleanParam(name: 'RUN', defaultValue: true, description: 'SELECT TO RUN')
    }
    triggers {
        pollSCM
    }
    tools{
        maven  'm3'
        jdk 'jdk11'
    }

    stages {
        stage('Clean Ws') {
            steps{
                cleanWs()
            }
        }
        stage('Git CheckoutOut'){
            steps{
                checkout scm
            }
        }
        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }
       stage('Static Code Analaysis') {
            steps {
                withSonarQubeEnv(credentialsId: 'sonar_token', installationName: 'sonarcloud') {
                    sh 'mvn verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar'
                }
            }
        }
         stage('Unit Test') {
            steps {
                sh 'mvn test'
            }
        }

         stage('Package') {
            steps {
                sh 'mvn package'
            }
        }
         
        stage ('publish to artifactory')  {

            steps {

            rtMavenResolver (
            id: 'resolver',
            serverId: 'artifactory1',
            releaseRepo: 'libs-release',
            snapshotRepo: 'libs-snapshot'
             )  
 
            rtMavenDeployer (
            id: 'deployer1',
            serverId: 'artifactory1',
            releaseRepo: 'libs-release-local',
            snapshotRepo: 'libs-snapshot-local',
    // By default, 3 threads are used to upload the artifacts to Artifactory. You can override this default by setting:
            
    // Attach custom properties to the published artifacts:
            properties: ['version=v1', 'publisher=emis']
            )

            rtMavenRun (
            // Tool name from Jenkins configuration.
            tool: 'm3',
            pom: 'pom.xml',
            goals: 'clean install',
            // Maven options.
            opts: '-Xms1024m -Xmx4096m',
            resolverId: 'resolver',
            deployerId: 'deployer1',
            // If the build name and build number are not set here, the current job name and number will be used:
            buildName: 'my-build-name',
            buildNumber: '17',
            // Optional - Only if this build is associated with a project in Artifactory, set the project key as follows.
            project: 'my-project-key'
            )
            }
        }

    post {
        always {
            echo 'ALL GOOD '
        }
    }


}
