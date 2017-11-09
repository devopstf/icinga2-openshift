#!/usr/bin/env groovy
/* general vars */
def dockerRelasesRepo = "docker-releases"
def dockerSnapshotsRepo = "docker-snapshot"

def ansibleDockerImage = dockerRelasesRepo
def dockerPrivilegedArgs = '--privileged -v /var/run/docker.sock:/var/run/docker.sock --group-add docker'

backendFolder = 'openshift-icinga2'
backendDbFolder = backendFolder 

devopsFolder = 'devops'


pipeline {
	agent { label 'docker' }

    stages {
        stage('Init') {
            steps {
            	script {
					deployOpenshift = (params.deployOpenshift != null) ? params.deployOpenshift : false
					openshiftToken = (params.openshiftToken != null) ? params.openshiftToken : ""
					openshiftUrl = (params.openshiftUrl != null) ? params.openshiftUrl : "https://master.com"
					openshiftProjectName = (params.openshiftProjectName != null) ? params.openshiftProjectName : ""
					redeployOpenshiftTemplate = (params.redeployOpenshiftTemplate != null) ? params.redeployOpenshiftTemplate : ""
					openshiftPersistenceCore = (params.openshiftPersistenceCore != null) ? params.openshiftPersistenceCore : ""
					openshiftPersistenceFrontend = (params.openshiftPersistenceFrontend != null) ? params.openshiftPersistenceFrontend : ""
				}
			}
        }
         stage('Update: Monitoring Services') {
          
            steps {

					ansiblePlaybook(
				        playbook:  devopsFolder + '/ansible/monitoring-openshift-update.yml',
						credentialsId: 'credJenkins',
				        inventory: devopsFolder + '/ansible/inventory/os3dev',
						extraVars: [
						   docker_folder_core: openshiftPersistenceCore,
						   docker_folder_frontend: openshiftPersistenceFrontend ]
					)   
					   
				
            }
        }
        stage('Deploy: BBDD in Openshift') {
          
            steps {
				script {
					sh 'ansible-playbook ' + devopsFolder + '/ansible/mysql-openshift-deploy.yml ' +
					   ' -i ' + devopsFolder + '/ansible/inventory/os3dev ' +
					   ' -e openshift_token=' + openshiftToken +
					   ' -e openshift_url=' + openshiftUrl +
					   ' -e openshift_project_name=' + openshiftProjectName +
					   ' -e redeploy_openshift_template=' + redeployOpenshiftTemplate
				}
            }
        }
   	            
        stage('Deploy: Core in Openshift') {
          
            steps {
				script {
					sh 'ansible-playbook ' + devopsFolder + '/ansible/core-openshift-deploy.yml ' +
					   ' -i ' + devopsFolder + '/ansible/inventory/os3dev ' +
					   ' -e openshift_token=' + openshiftToken +
					   ' -e openshift_url=' + openshiftUrl +
					   ' -e openshift_project_name=' + openshiftProjectName +
					   ' -e redeploy_openshift_template=' + redeployOpenshiftTemplate
				}
            }
        }
         stage('Deploy: Frontend in Openshift') {
          
            steps {
				script {
					sh 'ansible-playbook ' + devopsFolder + '/ansible/frontend-openshift-deploy.yml ' +
					   ' -i ' + devopsFolder + '/ansible/inventory/os3dev ' +
					   ' -e openshift_token=' + openshiftToken +
					   ' -e openshift_url=' + openshiftUrl +
					   ' -e openshift_project_name=' + openshiftProjectName +
					   ' -e redeploy_openshift_template=' + redeployOpenshiftTemplate
				}
            }
        }
       
    }
	post {
		always{
			step([$class: 'WsCleanup', patterns: [[pattern: '**/target/**', type: 'INCLUDE']], cleanWhenFailure: false])
		}
		failure {
			emailext (
			   subject: "[Jenkins] FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
			   body: """<p>FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
			     <p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
			   mimeType: 'text/html',
			   recipientProviders: [[$class: 'DevelopersRecipientProvider']]
			 )
		}
	}
}
