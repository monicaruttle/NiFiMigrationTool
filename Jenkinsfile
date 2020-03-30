pipeline {
    agent any
    parameters {
        string(name: 'NIFI_SERVER_URL', defaultValue: 'http://localhost:8080', description: 'NiFi server to upload templates to')
        booleanParam(name: 'REPLACE_CONFLICTS', defaultValue: true, description: 'Replace any templates that have a name conflict with the one being uploaded')
        string(name: 'TEMPLATES', defaultValue: '', description: 'Comma-separated list of filenames in templates directory to upload. If empty string, then all are uploaded.')
    }

    stages {
        stage('Upload NiFi Templates') {
            steps {
                script {
                    echo "Uploading templates to ${params.NIFI_SERVER_URL}..."

                    echo "Retrieving root process group ID"
                    def rootProcessGroupInfo = readJSON text: sh(script: "curl ${params.NIFI_SERVER_URL}/nifi-api/flow/process-groups/root", returnStdout: true)
                    def rootProcessGroupId = rootProcessGroupInfo['processGroupFlow']['id']

                    if (params.TEMPLATES == '') {
                        def files = findFiles(glob: 'templates/*.xml')
                    } else {
                        def files = []
                        params.TEMPLATES.tokenize(',').each{ template_filename ->
                            files.addAll(findFiles(glob: "templates/${template_filename}"))
                        }
                    }

                    files.each { template_filename ->
                        echo "Attempting to upload $template_filename"

                        def uploadTemplateCmd = "curl -w '.%{http_code}' -F template=@$template_filename ${params.NIFI_SERVER_URL}/nifi-api/process-groups/${rootProcessGroupId}/templates/upload"
                        def response = sh(script: "${uploadTemplateCmd}", returnStdout: true)
                        def responseMessage = response.tokenize('.').first()
                        def responseCode = response.tokenize('.').last()

                        if (responseCode == '409' && params.REPLACE_CONFLICTS) {
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
                        } else if (responseCode == '409') {
                            echo "A template with the same name as $template_filename already exists, and replacing duplicates is set to false. Skipping template."
                        }else if (responseCode != '201') {
                            error "Received HTTP response code ${responseCode} when uploading $template_filename, response from server:\n${responseMessage}"
                        }
                    }
                }
            }
        }
    }
}