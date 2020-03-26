pipeline {
    agent any
    parameters {
        string(name: 'NIFI_SERVER_URL', defaultValue: 'http://localhost:8080', description: 'NiFi server to upload templates to')
    }

    stages {
        stage('Upload NiFi Templates') {
            steps {
                script {
                    echo "Uploading templates to ${params.NIFI_SERVER_URL}..."

                    def files = findFiles(glob: 'templates/*.xml')
                    files.each {
                        echo "Attempting to upload $it"

                        def uploadTemplateCmd = "curl -w '.%{http_code}' -F template=@$it ${params.NIFI_SERVER_URL}/nifi-api/process-groups/07b8ebc9-0171-1000-0bc7-44ab34f61036/templates/upload"
                        def response = sh(script: "${uploadTemplateCmd}", returnStdout: true)
                        def responseMessage = response.tokenize('.').first()
                        def responseCode = response.tokenize('.').last()

                        if (responseCode == '409') {
                            // need to delete the existing template before uploading the new one.
                            def templateName = responseMessage.replace("A template named '",'')
                            templateName = templateName.replace("' already exists",'')

                            echo "Replacing ${templateName}..."
                            
                            // delete the template with the same name as the one being uploaded
                            def existingResources = readJSON text: sh(script: "curl -v ${params.NIFI_SERVER_URL}/nifi-api/resources", returnStdout: true)
                            existingResources = existingResources['resources']
                            existingResources.each { resource ->
                                if (resource['name'] == templateName && resource['identifier'].contains('templates')) {
                                    echo "Deleting existing ${templateName}..."
                                    def templateToDelete = resource['identifier']
                                    sh "curl -v -X DELETE ${params.NIFI_SERVER_URL}/nifi-api${templateToDelete}"
                                }
                            }

                            // re-upload the template with the same name
                            echo "Uploading ${templateName} again..."
                            response = sh(script: "${uploadTemplateCmd}", returnStdout: true)
                            responseCode = response.tokenize('.').last()
                            
                            if (responseCode != '201') {
                                responseMessage = response.tokenize('.').first()
                                error "Received HTTP response code ${responseCode} when uploading ${templateName}, response from server:\n${responseMessage}"
                            }
                        } else if (responseCode != '201') {
                            error "Received HTTP response code ${responseCode} when uploading $it, response from server:\n${responseMessage}"
                        }
                    }
                }
            }
        }
    }
}