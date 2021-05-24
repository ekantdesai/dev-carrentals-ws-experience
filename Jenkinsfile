pipeline {
  agent any

  environment {
	  mavenBuildType = "${readMavenPom().getVersion().contains('RELEASE') ? "release" : "snapshot"}"
	  groupId = readMavenPom().getGroupId().replaceAll('\\.','/')
	  artifactId = readMavenPom().getArtifactId()
	  artifactVersion = readMavenPom().getVersion()
	  artifactVersionPath = readMavenPom().getVersion()
	  artifactName = readMavenPom().getPackaging()

  }
  stages {
    stage('Initialize') {
        steps {
          script {
            // Skips the first build to load parameters from Jenkinsfile.
            // if (!currentBuild.getPreviousBuild()) {
            //   echo "First-ever build attempt. Will only refresh and consequently be able to give you a \"Build with Parameters\" option after this..."
            //   currentBuild.result = 'UNSTABLE'
            // } else 
            if (mavenBuildType == 'release' && env.GIT_BRANCH != 'origin/master') {
              echo "You're trying to do a release build from a non-master" + " (i.e, " + env.GIT_BRANCH + ") " + "branch. We don't do that..."
              currentBuild.result = 'UNSTABLE'
            }
          }
        }
     }
	stage('Unit Test') {
	  when {
          expression { currentBuild.result != 'UNSTABLE' }
      }
      steps {
        bat 'mvn clean test'
      }
    }
	stage('Build Artifact') {
	  when {
          expression { currentBuild.result != 'UNSTABLE' }
      }
      steps {
		bat 'mvn package'
		bat "ren target\\${artifactId}-${artifactVersion}-${artifactName}.jar ${artifactId}-${artifactVersionPath}-${artifactName}.jar"
      }
    }
	stage('Upload Artifact') {
	  when {
          expression { currentBuild.result != 'UNSTABLE' }
      }
	  steps {
	  //rtDownload (
			//serverId: 'jenkins-artifactory-server',
			//spec: '''{
			//	  "files": [
			//		{
			//		  "pattern": "libs-snapshot-local/com/mycompany/maventest/1.0.0-SNAPSHOT/",
			//		  "target": "snapshot-app/"
			//		}
			//	 ]
			//}''',
		 
			// Optional - Associate the uploaded files with the following custom build name and build number,
			// as build artifacts.
			// If not set, the files will be associated with the default build name and build number (i.e the
			// the Jenkins job name and number).
			//buildName: 'AppTest',
			//buildNumber: '1'
		//)
		echo "upload artifact into " + "libs-${mavenBuildType}-local/${groupId}/${artifactId}/${artifactVersionPath}/"
		rtUpload (
			serverId: 'jenkins-artifactory-server',
			spec: '''{
				  "files": [
					{
					  "pattern": "target/${artifactId}-${artifactVersionPath}-${artifactName}.jar",
					  "target": "conflowence-libs-${mavenBuildType}-local/${groupId}/${artifactId}/${artifactVersionPath}/"
					}
				 ]
			}''',
			//"pattern": "target/maventest*.jar",
			//"target": "libs-snapshot-local/com/mycompany/maventest/1.0.0-SNAPSHOT/"
			// Optional - Associate the uploaded files with the following custom build name and build number,
			// as build artifacts.
			// If not set, the files will be associated with the default build name and build number (i.e the
			// the Jenkins job name and number).
			buildName: 'AppTest',
			buildNumber: '1'
		)
	  }
	}
    stage('Deploy To Cloudhub') {
      when {
          expression { currentBuild.result != 'UNSTABLE' }
      }
      steps {
        bat 'mvn clean package deploy -DmuleDeploy'
      }
    }
  }
}
