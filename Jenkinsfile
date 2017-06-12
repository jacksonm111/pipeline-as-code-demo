#!groovy

//input message: '', parameters: [[$class: 'GitParameterDefinition', branchFilter: '.*', defaultValue: '*/master', name: 'tag', tagFilter: '*', type: 'PT_BRANCH_TAG', Description: '']]
//def res = input (message: '', parameters: [[$class: 'TextParameterDefinition', id: 'res', name: 'tag', defaultValue: '*/master', Description: '']])

pipeline {
  environment {
    def tag="master"
    def LABEL_LOWER="label-intg"
    def LABEL_UPPER="label-prod"
    def url='https://github.com/jacksonm111-org/pipeline-as-code-demo.git'
    def TIME1=5
    def TIME2=10
  }
  agent {label 'label-intg'}

  stages {
    /*run on master*/
    /*dump env*/
    stage ('dump-env')
    {
      agent { label 'master' }
      steps {
        sh 'env > env.txt'
        script {
          s = readFile('env.txt').split("\r?\n")
          for(i=0; i <s.length; i++) {
//        println s[i]
          }
        }

        /*checkpoint and clean workspace*/
        stash('Before cleanup')
        step([$class: 'WsCleanup'])
        
        /*checkout*/
//        checkout scm:([$class: 'GitSCM', branches: [[name: "*/${tag}"]], doGenerateSubmoduleConfigurations: true
//        , extensions: [], gitTool: 'Default', submoduleCfg: [], userRemoteConfigs: [[credentialsId: "${CREDS}"
//        , url: "${url}"]]])
checkout scm
        stash('checkout')    
      }
    }
    
    /*run on slave*/
    stage ('Dev') {
      steps {
        unstash('checkout')
        /*build*/
        sh "${tool 'M3'}/bin/mvn clean package"
        /*deploy*/
        dir('target') {stash name: 'war', includes: 'x.war'}
      }
    }

    /*run on slave*/
    stage ('QA') {
      steps {
        parallel(
          longerTests: {
            sh "ls -la"
            sh "sleep ${TIME1}"
          } 
          , quickerTests: {
            sh "ls -la"
            sh "sleep ${TIME2}"
          }
        )
      }
    }

    /*run on slave*/
    stage ('Staging') {
      steps {
        unstash 'war'
        sh "scp x.war /home/ubuntu/prod/jboss-as-7.1.1.Final/standalone/deployments/staging.war"
      }
    }

    /*run on master*/
    stage('Notify approver')
    {
      agent { label 'master' }
      steps {
        mail body: "${JOB_URL}", subject: "Request deploy to production (Build ${BUILD_ID})", to: 'hartmanph@nih.gov'
      }
    }
    stage('Wait for approval')
    {
      agent { label 'master' }
      steps {
        timeout(time: 1, unit: 'MINUTES'){
          input message: 'Deploy to prod?'/*, submitter: 'admin'*//*uncomment when authentication enabled*/
        }
      }
    }

    /*run on slave*/
    stage ('Production') {
      agent { label 'label-prod' }
      steps {
        echo 'Production server looks to be alive'
        unstash 'war'
        sh "scp x.war /home/ubuntu/prod/jboss-as-7.1.1.Final/standalone/deployments/production.war"
        echo "Deployed to production"
      }
    }
  }
}
