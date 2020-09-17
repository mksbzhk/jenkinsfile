node {
	stage('Pull'){
		withEnv(['PATH+EXTRA=/usr/sbin:/usr/bin:/sbin:/bin']){
			sh 'rm -rf $(pwd)/*'
			sh 'rm -rf .git'
		}
		checkout([$class: 'GitSCM', branches: [[name: '${GIT_TAG_BRANCH}']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'CloneOption', noTags: false, reference: '', shallow: false]], submoduleCfg: [], userRemoteConfigs: [[url: '${GIT_REPO_SSH_URL}']]])
	}

	stage('Validate'){
		withEnv(['PATH+EXTRA=/usr/sbin:/usr/bin:/sbin:/bin']){
			sh script: '''#!/bin/bash
			env | grep AWS > dockerenv
			env | grep "TF_VAR_" >> dockerenv'''

			docker.image('hashicorp/terraform:0.12.21').inside('--env-file ${WORKSPACE}/dockerenv --entrypoint=""'){
				sh script: '''#!/bin/sh
				terraform --version
				terraform init -backend-config="access_key=$AWS_STATE_ACCESS_KEY_ID" -backend-config="secret_key=$AWS_STATE_SECRET_ACCESS_KEY" -backend-config="region=$AWS_STATE_DEFAULT_REGION" -backend-config="bucket=$AWS_STATE_BUCKET"
				terraform validate'''
			}
		}
	}

	stage('Plan'){
		withEnv(['PATH+EXTRA=/usr/sbin:/usr/bin:/sbin:/bin']){
			docker.image('hashicorp/terraform:0.12.21').inside('--env-file ${WORKSPACE}/dockerenv --entrypoint=""'){
				sh script: '''#!/bin/sh
				terraform plan -out planfile'''
			}
		}
	}

	stage('Apply'){

		timeout(time: 15, unit: "MINUTES") {
			input message: 'Proceed to apply?', ok: 'Yes'
		}


		withEnv(['PATH+EXTRA=/usr/sbin:/usr/bin:/sbin:/bin']){
			docker.image('hashicorp/terraform:0.12.21').inside('--env-file ${WORKSPACE}/dockerenv --entrypoint""'){
				sh script: '''#!/bin/sh
				terraform apply -input=false "planfile"'''
			}
		}
	}
}
