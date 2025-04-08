import groovy.json.JsonSlurper
import groovy.json.JsonSlurperClassic
pipeline {
    agent any

    environment {
        ORCHESTRATOR_URL = 'https://cloud.uipath.com/ltimileeikdf/DefaultTenant/orchestrator_'
        TENANT_NAME = 'DefaultTenant'
        CLIENT_ID = '8DEv1AMNXczW3y4U15LL3jYf62jK93n5'
        USER_KEY = 'I4s7rahv-OsO28bnLWdVV5sAekXibC4w-veW6Lveq5lqY'
        FOLDER_NAME = 'Testing' // Change as per your folder structure
        PROCESS_NAME = 'UiPath_CICD_Integration'
    }
      triggers {
        githubPush()
    }
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/mangeshdhotre-2/UiPathCICDIntegration.git'
            }
        }

        stage('Build UiPath Package') {
            steps {
                script {
                  bat "\"C:\\Users\\Admin\\AppData\\Local\\Programs\\UiPath\\Studio\\UiPath.Studio.CommandLine.exe\" publish -p \"C:\\Users\\Admin\\Documents\\UiPath\\UiPath_CICD_Integration\\project.json\""
                }
            }
        }

        stage('Deploy to Orchestrator') {
            steps {
                script {
                    def authResponse = httpRequest(
                        url: "https://account.uipath.com/oauth/token",
                        httpMode: 'POST',
                         customHeaders: [
                             [name: 'Content-Type', value: 'application/x-www-form-urlencoded'],
                            [name: 'X-UIPATH-TenantName', value: 'DefaultTenant']
                             ],
                        requestBody: "grant_type=refresh_token&client_id=${CLIENT_ID}&refresh_token=${USER_KEY}"
                    )
                    echo "Token Created"
                    def jsonResponse = new JsonSlurper().parseText(authResponse.content)  // FIXED
                    def token = jsonResponse.access_token
                    echo "Token:${token}"
                    def uploadResponse = httpRequest(
                        url: "${ORCHESTRATOR_URL}/odata/Processes/UiPath.Server.Configuration.OData.UploadPackage",
                        httpMode: 'POST',
                        customHeaders: [ 
                            [name: 'Content-Type', value: 'application/x-www-form-urlencoded'],
                            [name: 'Authorization', value: "Bearer ${token}"],
                            [name: 'X-UIPATH-OrganizationUnitId', value: '6269096']],
                        multipartName: 'file',
                        uploadFile: 'C:\\ProgramData\\UiPath\\Packages\\UiPath_CICD_Integration.1.0.12.nupkg'
                    )
                     echo "📦 Raw Response: ${uploadResponse.content}"
                    

                   
                    


                }
            }
        }

        
    }
}

