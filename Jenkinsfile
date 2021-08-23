pipeline {
  agent any

  environment {
        NPM_TOKEN               = credentials('npm-token')
        AWS_ACCESS_KEY_ID       = credentials('aws-access-key-id')
        AWS_SECRET_ACCESS_KEY   = credentials('aws-secret-access-key')
        GITHUB_CREDENTIALS      = credentials('github-jenkins')
        DATADOG_API_KEY         = credentials('datadog-api-key')
        SONARQUBE_USER          = credentials('sonarqube-user')
  }

  stages {

    stage('Setup Dependencies') {
        steps {

            // create a file so that we can access our private npm repos
            writeFile file: ".npmrc", text: "//registry.npmjs.org/:_authToken=${NPM_TOKEN}"

            // this will install all of the dependencies
            // sh 'yarn install'
             sh 'npm install'
            // set the version number equal to the build number so that everything lines up nicely
            sh 'npm version "6.9.0.${BUILD_ID}" --force'
        }
    }

    stage('Dev Deploy'){
        steps{
            deployToEnvironment('dev')
        }
    }
    stage('Cleanup') {
        steps {

            // print all of the environment variables
            sh 'printenv'

            sh 'git config --global credential.helper store'
            sh 'git config --global user.name "${GITHUB_CREDENTIALS_USR}"'
            sh 'git config --global user.password "${GITHUB_CREDENTIALS_PASS}"'
            sh 'git tag ${BUILD_TAG}'
            sh 'git push --tags'
        }
    }
  }//stages

}

//global variables
DEPLOY_START = null
DEPLOY_END = null
DEPLOY_DURATION_SECONDS = null

//functions
def deployToEnvironment(environment)
{
    DEPLOY_START = (new Date()).getTime()

    try
    {
        sh "sls deploy --stage ${environment}"

            DEPLOY_END = (new Date()).getTime()
            DEPLOY_DURATION_SECONDS = (DEPLOY_END - DEPLOY_START) / 1000

    }
    catch (err)
    {

        throw err;
    }
}

def sendDataDogEvent(title, text, priority, tags, alertType)
{
    sh """curl  -X POST -H \"Content-type: application/json\" \
        -d \'{ \n \
                \"title\" : \"${title}\", \
                \"text\": \"${text}\", \
                \"priority\": \"${priority}\", \
                \"tags\": ${tags}, \
                \"alert_type\": \"${alertType}\", \
                \"source_type_name\": \"jenkins\" \
            }\' \
        \"https://api.datadoghq.com/api/v1/events?api_key=${DATADOG_API_KEY}\" """
}
