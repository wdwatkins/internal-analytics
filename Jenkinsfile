pipeline {
  agent {
        node {
            label 'team:makerspace'
        }
    }
    environment {
     GA_AUTH_FILE = credentials('GOOGLE_ANALYTICS_SERVICE_JSON')   
     GA_CLIENTID_FILE = credentials('GOOGLE_ANALYTICS_CLIENTID_JSON')
    }
    parameters {
        gitParameter name: 'BRANCH_TAG',
                     type: 'PT_BRANCH_TAG',
                     defaultValue: 'master'
        choice(choices: ['test', 'prod'], description: 'Tier to deploy to', name: 'TIER')
    }
    triggers {
        cron('H 9 * * *')
    }
  stages {
    stage('Clean Workspace') {
      steps{
        cleanWs()
      }
    }
    stage('Checkout repo and pull cache from S3') {
      steps {
        sh 'wget -O DOIRootCA2.cer http://sslhelp.doi.net/docs/DOIRootCA2.cer'
        checkout([$class: 'GitSCM',
                          branches: [[name: "${params.BRANCH_TAG}"]],
                          doGenerateSubmoduleConfigurations: false,
                          extensions: [],
                          gitTool: 'Default',
                          submoduleCfg: [],
                          userRemoteConfigs: [[url: 'https://github.com/usgs-makerspace/analytics_pipeline']]
                        ])
        //sh 'aws s3 sync s3://wma-analytics-data/dashboard/${TIER}/cache/ cache/'
      }
    }
    stage('pull Google Analytics data') {
      agent {
        docker {
          image 'code.chs.usgs.gov:5001/wma/iidd/vizlab-internal-analytics:latest'
          registryUrl 'https://code.chs.usgs.gov:5001/wma/iidd/vizlab-internal-analytics'
          registryCredentialsId 'jenkins_ci_access_token'
          alwaysPull true
          reuseNode true
          label 'team:makerspace'
        } 
      }
      steps {
        retry(2){
          sh 'Rscript -e "library(vizlab); createProfile(directory='vizlab'); unpublish(); vizmake()"'
        }
      }
    }
  }
      post {
        unstable {
            mail to: 'mhines@usgs.gov, wwatkins@usgs.gov',
            subject: "${TIER} Unstable: ${currentBuild.fullDisplayName}",
            body: "Pipeline is unstable ${env.BUILD_URL}"
        }
        failure {
            mail to: 'mhines@usgs.gov, wwatkins@usgs.gov',
            subject: "${TIER} Failure: ${currentBuild.fullDisplayName}",
            body: "Pipeline failed ${env.BUILD_URL}"
        }
        changed {
            mail to: 'mhines@usgs.gov, wwatkins@usgs.gov',
            subject: "${TIER} Changes: ${currentBuild.fullDisplayName}",
            body: "Pipeline detected changes ${env.BUILD_URL}"
        }
    } 
}
