#!groovy

//input message: '', parameters: [[$class: 'GitParameterDefinition', branchFilter: '.*', defaultValue: '*/master', name: 'tag', tagFilter: '*', type: 'PT_BRANCH_TAG', Description: '']]
def res = input (message: '', parameters: [[$class: 'TextParameterDefinition', id: 'res', name: 'tag', defaultValue: '*/master', Description: '']])
try {
    checkpoint('Before Dev')
} catch (NoSuchMethodError _) {
    echo 'Checkpoint feature available in CloudBees Jenkins Enterprise.'
}
echo '10'
def tag = res['name']
echo '11'
echo ("tag=${tag}")
echo '13'

stage 'Dev'
node ('lower-test') {
    checkout([$class: 'GitSCM', branches: [[name: "ref/heads/${tag}"]], doGenerateSubmoduleConfigurations: false, extensions: [], gitTool: 'Default', submoduleCfg: [], userRemoteConfigs: [[credentialsId: '4602578b-8ff4-425a-a487-a9bfdc6354c8', url: 'https://github.com/jacksonm111/pipeline-as-code-demo.git']]])    
    mvn 'clean package'
    dir('target') {stash name: 'war', includes: 'x.war'}
}
echo '21'

stage 'QA'
parallel(longerTests: {
    runTests(30)
}, quickerTests: {
    runTests(20)
})

stage name: 'Staging', concurrency: 1
node ('lower-test') {
    deploy 'staging'
}

input message: "Does staging look good?"
try {
    checkpoint('Before production')
} catch (NoSuchMethodError _) {
    echo 'Checkpoint feature available in CloudBees Jenkins Enterprise.'
}

stage name: 'Production', concurrency: 1
node ('upper-tier'){
    echo 'Production server looks to be alive'
    deploy 'production'
    echo "Deployed to production"
}

def mvn(args) {
    sh "${tool 'M3'}/bin/mvn ${args}"
}

def runTests(duration) {
    node {
        sh "sleep ${duration}"
        }
    }

def deploy(id) {
    unstash 'war'
    sh "cp x.war /tmp/${id}.war"
}

def undeploy(id) {
    sh "rm /tmp/${id}.war"
}
