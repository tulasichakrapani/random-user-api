properties([pipelineTriggers([githubPush()])])

pipeline {
  agent any
  environment {
    BUILD_VERSION = "${currentBuild.number}.0.0"
    ANYPOINT_CREDENTIALS = credentials('Anypoint_Sandbox_Creds')
  }

  tools {
    maven 'M3' //defines maven tool (configured in "Global Configuration Tool -> Maven -> Maven Installations")
  }

  stages {
    stage('Build and Test') {
      steps {
        script {
          //Use Pipeline Utility Steps plugin to read information from pom.xml into env variables
          releaseVersion = readMavenPom().getVersion()
          if (!releaseVersion.endsWith('-SNAPSHOT')) {
            throw new Exception("Version does not correspond to SNAPSHOT, please add the -SNAPSHOT suffix in the version")
          }
          echo "Starting Build and Test..."
          //gets the settings.xml file from Managed files where the id of the file is 54072478-e589-4704-b917-580b2c515dde
          configFileProvider([configFile(fileId: '54072478-e589-4704-b917-580b2c515dde', targetLocation: 'settings.xml', variable: 'MAVEN_SETTINGS_XML')]) {
            sh "mvn -s $MAVEN_SETTINGS_XML -Dmaven.test.failure.ignore clean verify"
          }
          echo "Build and Test: ${currentBuild.currentResult}"
        }
      }
      post {
        success {
          echo "...Build and Test Succeeded"
        }
        unsuccessful {
          echo "...Build and Test Failed"
        }
      }
    }

    stage('Deploy Snapshot to Artifactory') {
      steps {
        script {
          echo "Starting Deploy Snapshot Artifact..."
          //gets the settings.xml file from Managed files where the id of the file is 54072478-e589-4704-b917-580b2c515dde
          configFileProvider([configFile(fileId: '54072478-e589-4704-b917-580b2c515dde', targetLocation: 'settings.xml', variable: 'MAVEN_SETTINGS_XML')]) {}
          echo "Artifact Deployed: ${currentBuild.currentResult}"
        }
      }
      post {
        success {
          echo "...Deploy Artifact Succeeded for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
        }
        unsuccessful {
          echo "...Deploy Artifact Failed for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
        }
      }
    }

    stage('Deploy to Sandbox environment') {
      steps {
        script {
          echo "Starting Sandbox environment"

          applicationName = 'sandbox-' + readMavenPom().getArtifactId()
          echo "applicationName=${applicationName}"
          //retrieves the properties from the deploy.properties file stored in the repo
          props = readProperties(file: 'deploy.properties')
          anypointMuleVersion = props['anypoint.mule.version']
          anypointMuleEnvironment = props['anypoint.mule.environment']
          businessGroupId = props['anypoint.mule.businessGroupId']
          persistentQueues = props['cloudhub.persistentQueues']
          cloudhubEnv = props['anypoint.mule.environment']
          cloudhubWorkerType = props['cloudhub.workerType']
          cloudhubWorkers = props['cloudhub.workers']
          reqAppCoverage = props['reqAppCoverage']
          reqResourceCoverage = props['reqResourceCoverage']
          reqFlowCoverage = props['reqFlowCoverage']
          failBuild = props['failBuild']
          echo "anypointMuleVersion=${anypointMuleVersion}"
          echo "anypointMuleEnvironment=${anypointMuleEnvironment}"
          echo "businessGroupId=${businessGroupId}"
          echo "persistentQueues=${persistentQueues}"
          echo "cloudhubEnv=${cloudhubEnv}"
          echo "cloudhubWorkerType=${cloudhubWorkerType}"
          echo "cloudhubWorkers=${cloudhubWorkers}"
          echo "Deploy to Sandbox: ${currentBuild.currentResult}"

          configFileProvider([configFile(fileId: '54072478-e589-4704-b917-580b2c515dde', targetLocation: 'settings.xml', variable: 'MAVEN_SETTINGS_XML')]) {
            // Run the maven build
            sh ""
            " mvn clean package deploy -DmuleDeploy -U --batch-mode -s $MAVEN_SETTINGS_XML \
                        -Danypoint.platform.config.analytics.agent.enabled=true \
                        -Dapp.runtime=${anypointMuleVersion}  \
                        -Dcloudhub.application.name=${applicationName} \
                        -Denvironment=${anypointMuleEnvironment} \
                        -Dbusiness.group.id=${businessGroupId} \
                        -Dcloudhub.workerType=${cloudhubWorkerType} \
                        -Dcloudhub.persistentQueues=${persistentQueues} \
                        -Dcloudhub.objectStoreV2=true \
                        -DattachMuleSources=true \
                        -Danypoint.platform.client_id=${env.ANYPOINT_CREDENTIALS_USR} \
                        -Danypoint.platform.client_secret=${env.ANYPOINT_CREDENTIALS_PSW} \
                        -DreqAppCoverage=${reqAppCoverage} \
                        -DreqResourceCoverage=${reqResourceCoverage} \
                        -DreqFlowCoverage=${reqFlowCoverage} \
                        -DfailBuild=${failBuild} \
                        -Dcloudhub.workers=${cloudhubWorkers} "
            ""
          }
        }
      }
      post {
        success {
          echo "...Deploy to Sandbox Succeeded for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
        }
        unsuccessful {
          echo "...Deploy to Sandbox Failed for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
        }
      }
    }
  }
  post {
    success {
      echo "All Good: ${env.BUILD_VERSION}"
    }
    unsuccessful {
      echo "Not So Good: ${env.BUILD_VERSION}"
    }
    always {
      echo "Pipeline result: ${currentBuild.result}"
      echo "Pipeline currentResult: ${currentBuild.currentResult}"
      cleanWs()
    }
  }
}