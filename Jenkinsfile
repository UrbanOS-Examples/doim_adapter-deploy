library(
    identifier: 'pipeline-lib@4.8.0',
    retriever: modernSCM([$class: 'GitSCMSource',
                          remote: 'https://github.com/SmartColumbusOS/pipeline-lib',
                          credentialsId: 'jenkins-github-user'])
)

properties([
    pipelineTriggers([scos.dailyBuildTrigger()]),
    parameters([
        booleanParam(defaultValue: false, description: 'Deploy to development environment?', name: 'DEV_DEPLOYMENT'),
        string(defaultValue: 'latest', description: 'Image tag to deploy to dev environment', name: 'DEV_IMAGE_TAG')
    ])
])

def applicationName = "doim-adapter"
def doStageIf = scos.&doStageIf
def doStageIfDeployingToDev = doStageIf.curry(env.DEV_DEPLOYMENT == "true")
def doStageIfMergedToMaster = doStageIf.curry(scos.changeset.isMaster && env.DEV_DEPLOYMENT == "false")
def doStageIfRelease = doStageIf.curry(scos.changeset.isRelease)

node ('infrastructure') {
    ansiColor('xterm') {
        scos.doCheckoutStage()

        doStageIfDeployingToDev('Deploy to Dev') {
            deployTo(applicationName, 'dev', "--set image.tag=${env.DEV_IMAGE_TAG} --recreate-pods")
        }

        doStageIfMergedToMaster('Deploy to Staging') {
            deployTo(applicationName, 'staging')
            scos.applyAndPushGitHubTag('staging')
        }

        doStageIfRelease('Deploy to Production') {
            deployTo(applicationName, 'prod')
            scos.applyAndPushGitHubTag('prod')
        }
    }
}

def deployTo(applicationName, environment, extraArgs = '') {
    scos.withEksCredentials(environment) {
        sh("""#!/bin/bash
            helm upgrade --install ${applicationName} . \
                --namespace=doim \
                ${extraArgs}
        """.trim())
    }
}
