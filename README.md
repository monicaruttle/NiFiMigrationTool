# NiFi Migration Tool
Jenkins 2 Pipeline to automatically upload source controlled Apache NiFi templates to a configurable NiFi server.

This workflow keeps your NiFi templates in source control, and automates the deployment of them into a NiFi instance. This can be used as a means to promote your templates from one environment (STG) to another (PRD). It can also be used for completely migrating one instance to another, such as with a version upgrade.

## Usage
1) Checkout this repo and reset the remote to your own git repo. This will be where you store your NiFi templates.
2) There is an example template in the 'templates' directory. You can use this to test the pipeline, however that directory will be where your XML-formatted templates will live.
3) Follow these instructions to create a Jenkins job that builds a pipeline from SCM: https://jenkins.io/doc/book/pipeline/getting-started/#defining-a-pipeline-in-scm
4) Build your job with the NiFi server parameter set to the NiFi instance you would like your templates uploaded to. Specify if you would like to replace any conflicting templates with the same name, or skip them. You can also specify which template files to upload, however the default is to upload all of them.

## Note
- Templates must be manually downloaded and stored in the 'templates' directory from the source NiFi instance.
- Job must manually be built (i.e. it is not triggered by a merge to master).

## Getting Started with Development
To contribute to this pipeline, you'll need a NiFi and Jenkins instance running to test your changes. In order to simplify things, you can run them as Docker containers (Docker installation guide: https://docs.docker.com/get-started/#set-up-your-docker-environment)

### Run NiFi and Jenkins
```bash
docker run -d --name nifi -p 8080:8080 apache/nifi:latest
docker run -d --name jenkins -p 8081:8080 -v /var/jenkins_home jenkins/jenkins
```

You'll need the IP address of the NiFi container in order to deploy templates to it.
```bash
docker inspect <NiFi container name from docker run command> | jq '.[0].NetworkSettings.Networks.bridge.IPAddress'
```
  
## TODO
- Configure a source NiFi instance to automatically scrape all templates from.
