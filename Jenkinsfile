pipeline {
   agent{
       label "master"
   }

   tools {
       maven "Maven_Jenkins"
   }
    environment {

        // This can be nexus3 or nexus2

        NEXUS_VERSION = "nexus3"

        // This can be http or https

        NEXUS_PROTOCOL = "http"

        // Where your Nexus is running

        NEXUS_URL = "stefhannya-thinkcentre-m920q:8081"

        // Repository where we will upload the artifact

        NEXUS_REPOSITORY = "RepositorioPrueba"

        // Jenkins credential id to authenticate to Nexus OSS

        NEXUS_CREDENTIAL_ID = "nexus-credentials"

    }
   stages {
      stage('Clone') {
         steps {
             script{
                git "https://github.com/stefhannyaf/Pruebam.git" 
             }
         }
      }
      stage('mvn build'){
          steps{
              script{
                  sh "mvn package -DskipTests=true"
              }
          }
      }
	
      stage('SonarQube'){
		steps{
			script{
			  def scannerHome = tool 'sonar_scanner';
			  withSonarQubeEnv('SonarQube'){
				sh "${scannerHome}/bin/sonar-scanner -Dproject.settings=${scannerHome}/bin/sonar-project.properties"
				
			  }
			}
		}
      }
      stage('publicar a nexus'){
          steps{
              
                script {

                    // Read POM xml file using 'readMavenPom' step , this step 'readMavenPom' is included in: https://plugins.jenkins.io/pipeline-utility-steps

                    pom = readMavenPom file: "pom.xml";

                    // Find built artifact under target folder

                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");

                    // Print some info from the artifact found

                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"

                    // Extract the path from the File found

                    artifactPath = filesByGlob[0].path;

                    // Assign to a boolean response verifying If the artifact name exists

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

                                // Artifact generated such as .jar, .ear and .war files.

                                [artifactId: pom.artifactId,

                                classifier: '',

                                file: artifactPath,

                                type: pom.packaging],

                                // Lets upload the pom.xml file for additional information for Transitive dependencies

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
              }
          }
		stage('Desplegar'){
			steps{
				script{
				  sh 'cp target/Pruebam.war /var/lib/tomcat8/webapps/';
				}
			}
		}
      }
}
