#!groovy
import groovy.json.JsonSlurperClassic

node {
    // Define SFDX path
    def SFDX_CLI = "C:\\Program Files\\sf\\bin\\sfdx.cmd"

    // Define environment variables
    def SFDC_USERNAME = env.HUB_ORG_DH
    def SFDC_HOST = env.SFDC_HOST_DH
    def JWT_KEY_CRED_ID = env.JWT_CRED_ID_DH
    def CONNECTED_APP_CONSUMER_KEY = env.CONNECTED_APP_CONSUMER_KEY_DH

    println "🚀 Starting Jenkins Pipeline..."
    println "🔹 SFDX Path: ${SFDX_CLI}"
    println "🔹 Salesforce Username: ${SFDC_USERNAME}"
    println "🔹 Salesforce Instance URL: ${SFDC_HOST}"

    stage('Checkout Source') {
        println "🔄 Checking out source code..."
        checkout scm
        println "✅ Source code checkout complete."
    }

    withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_key_file')]) {
        stage('Authenticate Salesforce') {
            println "🔐 Authenticating with Salesforce..."
            def rc = bat returnStatus: true, script: """
                "${SFDX_CLI}" auth jwt grant ^
                --client-id ${CONNECTED_APP_CONSUMER_KEY} ^
                --jwt-key-file "${jwt_key_file}" ^
                --username ${SFDC_USERNAME} ^
                --set-default-dev-hub ^
                --instance-url ${SFDC_HOST}
            """
            if (rc != 0) { 
                println "❌ ERROR: Salesforce authentication failed!"
                error 'Salesforce authentication failed'
            }
            println "✅ Salesforce authentication successful."
        }

        stage('Convert Source Format') {
            println "🔄 Converting source format for deployment..."
            def convertStatus = bat returnStatus: true, script: """
                "${SFDX_CLI}" project convert source --output-dir deploy/
            """
            if (convertStatus != 0) { 
                println "❌ ERROR: Source conversion failed!"
                error 'Source conversion failed'
            }
            println "✅ Source conversion successful."
        }

        stage('Deploy Code') {
            println "🚀 Starting deployment to Salesforce..."
            def deployStatus = bat returnStatus: true, script: """
                "${SFDX_CLI}" project deploy start --metadata-dir deploy/ --target-org ${SFDC_USERNAME}
            """
            if (deployStatus != 0) { 
                println "❌ ERROR: Salesforce deployment failed!"
                error 'Salesforce deployment failed'
            }
            println "✅ Salesforce deployment successful."
        }
    }

    println "🎉 Jenkins Pipeline execution completed successfully!"
}
