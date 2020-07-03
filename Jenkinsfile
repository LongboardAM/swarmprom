pipeline {
	agent { label 'master' }
	environment {
		SERVICE="swarmprom"
		USERNAME = sh(script:'aws ssm get-parameter --name global_admin_username --with-decryption --query Parameter.Value --output text', returnStdout: true).trim()
		EMAIL = sh(script:'aws ssm get-parameter --name global_admin_email --with-decryption --query Parameter.Value --output text', returnStdout: true).trim()
		PASSWORD = sh(script:'aws ssm get-parameter --name global_admin_password --with-decryption --query Parameter.Value --output text', returnStdout: true).trim()
		DOMAIN="longboardinternal.com"
		SLACK_URL="https://hooks.slack.com/services/94BkKX5gYNgUwaXxwXzUBMLT"
		SLACK_CHANNEL="dockerswarm"
		SLACK_USER="SwarmAlertManager"
	}
	stages {
		stage("Create deploy") {
			environment {
		        USER_PASSWORD = sh(script:'htpasswd -nb "${USERNAME}" "${PASSWORD}" | sed -e s/\\\\$/\\\\$\\\\$/g', returnStdout: true).trim()
		    }
			steps {
				sh 'envsubst < "deploy.yml" > "production.yml"'
			}
		}
		stage("Deploy to stack") {
			steps {
				sshagent(credentials : ['master-cred']) {
					sh 'ssh ubuntu@${DOCKER_MAIN} "mkdir -p ~/${SERVICE}/deploy/${BUILD_NUMBER}/"'
					sh 'scp -rp ./ ubuntu@${DOCKER_MAIN}:~/${SERVICE}/deploy/${BUILD_NUMBER}/'
					sh 'ssh ubuntu@${DOCKER_MAIN} "cd ~/${SERVICE}/deploy/${BUILD_NUMBER} && docker stack deploy ${SERVICE} --compose-file production.yml --prune --with-registry-auth"'
				}
			}
		}
	}
	post {
		always {
			cleanWs()
		}
	}
}
