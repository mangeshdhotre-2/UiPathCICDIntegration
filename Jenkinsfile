pipeline {
    agent any

    environment {
        ORCHESTRATOR_URL = 'https://cloud.uipath.com/ltimileeikdf/DefaultTenant/orchestrator_/'
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
                    bat """
    \"C:\\Users\\Admin\\AppData\\Local\\Programs\\UiPath\\Studio\\UiPath.Studio.CommandLine.exe\" pack \"C:\\Users\\Admin\\Documents\\UiPath\\UiPath_CICD_Integration\\project.json\" -o \"C:\\Users\\Admin\\Documents\\UiPath\\UiPath_CICD_Integration\\Output\"
"""


                }
            }
        }

        stage('Deploy to Orchestrator') {
            steps {
                script {
                    def authResponse = httpRequest(
                        url: "https://account.uipath.com/oauth/token",
                        httpMode: 'POST',
                        contentType: 'application/json',
                         customHeaders: [
                            [name: 'X-UIPATH-TenantName', value: 'DefaultTenant']
                             ],
                        requestBody: "grant_type=refresh_token&client_id=${CLIENT_ID}&refresh_token=${USER_KEY}"
                    )

                    def token = readJSON(text: authResponse.content).access_token
                    echo "Token Created"
                    def uploadResponse = httpRequest(
                        url: "${ORCHESTRATOR_URL}/odata/Processes/UiPath.Server.Configuration.OData.UploadPackage",
                        httpMode: 'POST',
                        contentType: 'MULTIPART_FORM_DATA',
                        customHeaders: [[name: 'Authorization', value: "Bearer ${token}"],[name: 'X-UIPATH-OrganizationUnitId', value: '6269096']],
                        multipartName: 'file',
                        uploadFile: 'C:\\ProgramData\\Jenkins\\.jenkins\\workspace\\UiPathDemo_main\\Output\\UiPath_CICD_Integration.*.nupkg'
                    )

                    echo "Package Uploaded: ${uploadResponse.content}"
                }
            }
        }

        stage('Create New Process Version') {
            steps {
                script {
                    def releaseResponse = httpRequest(
                        url: "${ORCHESTRATOR_URL}/odata/Releases",
                        httpMode: 'GET',
                        customHeaders: [[name: 'Authorization', value: "Bearer ${token}"]],
                        contentType: 'APPLICATION_JSON'
                    )

                    def releases = readJSON(text: releaseResponse.content)
                    def releaseId = releases.value.find { it.ProcessKey == env.PROCESS_NAME }.Id

                    def updateResponse = httpRequest(
                        url: "${ORCHESTRATOR_URL}/odata/Releases(${releaseId})",
                        httpMode: 'PATCH',
                        customHeaders: [[name: 'Authorization', value: "Bearer ${token}"]],
                        contentType: 'APPLICATION_JSON',
                        requestBody: '{"InputArguments": "{}"}'
                    )

                    echo "Process Updated: ${updateResponse.content}"
                }
            }
        }
    }
}

