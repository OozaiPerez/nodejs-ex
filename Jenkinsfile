pipeline {
    agent none
    stages {
        // dev
        stage('[provison dev]'){
            agent { label 'jenkins-ansible-provision-nonprod' }
            stages {
                stage('provision dev') {
                    steps {
                        dir('resources') {
                            echo "provisioning OpenShift resources"

                            git url: 'git@bitbucket.org:employersappdev/devops-external-pipeline-poc.git',
                                    branch: 'master',
                                    credentialsId: 'jenkins'
                            sh 'ansible-playbook playbooks/provision-nodejs.yml --extra-vars "@roles/common/vars/nodejs-dev.yml"'
                        }
                        withCredentials([usernamePassword(credentialsId: 'jenkins-nonprod', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                            sh 'oc login https://openshift-master.ocpnonprod.eigwc.com --token=$PASSWORD'
                        }
                        // populate environment variables from secrets and configmap
                        echo "Configure application deployment config"
                        sh 'oc project nodejs-poc-nonprod'

                    }
                }
                //next stage
                stage('build and publish to Image Stream') {
                    steps {
                        withCredentials([usernamePassword(credentialsId: 'jenkins-nonprod', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                            sh 'oc login https://openshift-master.ocpnonprod.eigwc.com --token=$PASSWORD'
                        }
                        sh 'oc start-build nodejs'
                    }
                }

                stage('deploy-dev') {
                    steps {
                        echo "deploying to dev"
                        sh 'oc scale dc/nodejs --replicas=1'
                        sh 'oc rollout latest dc/nodejs'
                        sh 'oc rollout status --request-timeout=0 dc/nodejs || true'
                        sh 'oc rollout status --request-timeout=0 dc/nodejs'
                    }
                }

                stage('smoketest-dev') {
                    steps {
                        sh "APP_ENDPOINT=$(oc get endpoints nodejs -n nodejs | awk 'NR>1{print $2}')"

                        echo "smoke testing dev"
                        sh "curl $APP_ENDPOINT | grep -q -e 'HTTP/1.1 200 OK'"
                    }
                }

            }
        }

    }

}
