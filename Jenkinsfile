node {
	stage('Pull'){
		git branch: '${GIT_TAG_BRANCH#*/}', url: '${GIT_REPO_SSH_URL}'
	}

	stage('Validate'){
		withEnv(['PATH+EXTRA=/usr/sbin:/usr/bin:/sbin:/bin']){
			sh '''#!/bin/bash
			env | grep AWS > dockerenv
			env | grep ENV >> dockerenv'''

			docker.image('hashicorp/terraform:0.12.21').inside('--env-file ${WORKSPACE}/dockerenv'){
				sh '''#!/bin/bash
				terraform --version
				terraform init -backend-config="access_key=$STATE_AWS_ACCESS_KEY_ID" -backend-config="secret_key=$STATE_AWS_SECRET_ACCESS_KEY" -backend-config="region=$STATE_AWS_DEFAULT_REGION" -backend-config="bucket=$STATE_AWS_BUCKET"
				terraform validate'''
			}
		}
	}

	stage('Plan'){
		withEnv(['PATH+EXTRA=/usr/sbin:/usr/bin:/sbin:/bin']){
			docker.image('hashicorp/terraform:0.12.21').inside('--env-file ${WORKSPACE}/dockerenv'){
				sh '''#!/bin/bash
				terraform plan -out planfile'''
			}
		}
	}

	stage('Apply'){

		timeout(time: 15, unit: "MINUTES") {
			input message: 'Proceed to apply?', ok: 'Yes'
		}


		withEnv(['PATH+EXTRA=/usr/sbin:/usr/bin:/sbin:/bin']){
			docker.image('hashicorp/terraform:0.12.21').inside('--env-file ${WORKSPACE}/dockerenv'){
				sh '''#!/bin/bash
				terraform apply -input=false "planfile"'''
			}
		}
	}
}
