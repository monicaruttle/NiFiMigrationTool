# NiFi Migration Tool
Jenkins 2 Pipeline to automatically upload source controlled Apache NiFi templates to a configurable NiFi server.

This workflow keeps your NiFi templates in source control, and automates the deployment of them into a NiFi instance. This can be used as a means to promote your templates from one environment (STG) to another (PRD). It can also be used for completely migrating one instance to another, such as with a version upgrade.

## Usage
1) Checkout this repo and reset the remote to your own git repo. This will be where you store your NiFi templates.
2) There is an example template in the 'templates' directory. You can use this to test the pipeline, however that directory will be where your XML-formatted templates will live.
3) Follow these instructions to create a Jenkins job that builds a pipeline from SCM: https://jenkins.io/doc/book/pipeline/getting-started/#defining-a-pipeline-in-scm
4) Build your job with the NiFi server parameter set to the NiFi instance you would like your templates uploaded to.

## Note
- If a template with the same name already exists, it will get replaced.
- Templates must be manually stored in the 'templates' directory.
- Job must manually be built (i.e. it is not triggered by a merge to master).

## TODO
- Allow for configuring the conflict strategy when a template with the same name already exists (either replace or ignore).
- Configure a source NiFi instance to automatically scrape all templates from.
- Provide ability to specify a comma-separated list of templates to upload, rather than uploading all of them.
- Remove harcoded process group endpoint to upload templates to