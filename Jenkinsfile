properties([
  parameters([
		string(defaultValue: '', name: 'NUMERO_TICKET', description: 'NUMERO TICKET'),
		choice(name: 'AMBIENTE', choices: ['PRODUCCION', 'CERTIFICACION', 'SOPORTE', 'DESARROLLO'], description: 'AMBIENTE')
  ])
])

def notifySlack(String buildStatus = 'STARTED', Object servers) {
    // Build status of null means success.
    buildStatus = buildStatus ?: 'SUCCESS'

    def color

    if (buildStatus == 'STARTED') {
        color = '#D4DADF'
    } 
	else if (buildStatus == 'SUCCESS') {
        color = '#BDFFC3'
    } 
	else {
        color = '#FF9FA1'
    }

    def build_user = ''
    wrap([$class:'BuildUser']) {
        build_user = env.BUILD_USER
    }

    def msg = "${buildStatus}\nJob: `${env.JOB_NAME}`\nBuild Number: #${env.BUILD_NUMBER}:\nStarted by: ${build_user}\n${env.RUN_DISPLAY_URL}\nServers: ${servers}"

    slackSend(color: color, message: msg)
}


pipeline {
    agent any

    stages {
		stage('Inicio') {
			steps {
				script {
					echo "Iniciando Deploy del ticket " + ${NUMERO_TICKET} + " en el ambiente " + ${AMBIENTE}
					notifySlack('STARTED', servers)
				}
			}
		}

        stage('Pull del repositorio GIT') {
            steps {
                script {
                    withCredentials([sshUserPrivateKey(credentialsId: 'jenkins', keyFileVariable: 'SSH_KEYFILE', passphraseVariable: '', usernameVariable: 'SSH_USER')]) {
                        sh "ssh -i ${SSH_KEYFILE} -o StrictHostKeyChecking=no jenkins@perlnx07.unique-yanbal.com -C \'cd /Apps/test_cicd; git pull origin master;\'"
					}
                }
            }
        }

		stage('Ejecuto Pase') {
			steps {
				script {
                    withCredentials([sshUserPrivateKey(credentialsId: 'jenkins', keyFileVariable: 'SSH_KEYFILE', passphraseVariable: '', usernameVariable: 'SSH_USER')]) {
                        sh "ssh -i ${SSH_KEYFILE} -o StrictHostKeyChecking=no jenkins@perlnx07.unique-yanbal.com -C \'cd /Apps/test_cicd; cd ${NUMERO_TICKET}; cat pase.json\"'"
                    }
                }
			}
		} 
    }

	post {
		always {
			notifySlack(currentBuild.result, servers)
		}
	}
}