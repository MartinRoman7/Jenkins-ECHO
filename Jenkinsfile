def STAGE_STATUS
def REQUESTED_ACTION
def ROLLBACK

pipeline {
    agent any
    


    stages {
        stage('Connection to VM') {
            steps {
                sshagent(credentials : ['DigitalOcean_Servers']) {
                    sh 'ssh -o StrictHostKeyChecking=no root@<domain> uptime'
                    sh 'ssh root@<domain>'
                }
            }
        }
        stage('Create UAT Image') {
            steps {
                sshagent(credentials : ['DigitalOcean_Servers']) {
                    sh 'ssh -o StrictHostKeyChecking=no root@<domain> uptime'
                    sh 'ssh root@<domain> "/opt/Docker/Sistema_Pieenso/create_image.sh -i sistema-pieenso -c sistema-pieenso-preview"'
                }
            }
        }
        stage('Tag and Push UAT Image') {
            steps {
                sshagent(credentials : ['DigitalOcean_Servers']) {
                    sh 'ssh -o StrictHostKeyChecking=no root@<domain> uptime'
                    sh 'ssh root@<domain> "/opt/Docker/Sistema_Pieenso/push_image.sh"'
                }
            }
        }
        stage('Create UAT Container') {
            steps {
                sshagent(credentials : ['DigitalOcean_Servers']) {
                    sh 'ssh -o StrictHostKeyChecking=no root@<domain> uptime'
                    sh 'ssh root@<domain> "/opt/Docker/Sistema_Pieenso/create_container.sh -i <domain-harbor>:80/sistema_pieenso/sistema-pieenso:deploy -c sistema-pieenso-preview"'
                }
            }
        }
        stage('Check status UAT Container') {
            steps {
                script {
                    try {
                        sshagent(credentials : ['DigitalOcean_Servers']) {
                            sh 'ssh -o StrictHostKeyChecking=no root@<domain> uptime'
                            sh 'ssh root@<domain> "/opt/Monitoreo/ECHO_seminarios/status_container_preview.sh"'
                            catchError(buildResult: 'SUCCESS', stageResult: 'SUCCESS') {
                                STAGE_STATUS="SUCCESS"
                                sh 'exit 0'
                            }
                        }
                    } catch (err) {
                        catchError(buildResult: 'FAILURE', stageResult: 'FAILURE') {
                            STAGE_STATUS="FAILURE"
                            sh 'exit 1'
                        }
                    }
                }
            }
        }
        stage('Rollback - No Rollback') {
            steps {
                script {
                    if(STAGE_STATUS == 'FAILURE' ){
                        echo "RollBack"
                        sshagent(credentials : ['DigitalOcean_Servers']) {
                            sh 'ssh -o StrictHostKeyChecking=no root@<domain> uptime'
                            sh 'ssh root@<domain> "/opt/Docker/Sistema_Pieenso/roll_container.sh -i <domain-harbor>:80/sistema_pieenso/sistema-pieenso:productivo -c sistema-pieenso-preview"'
                            ROLLBACK="SUCCESS"
                        }
                    } else {
                        echo "No RollBack"
                    }
                }
            }
        }
        stage ('Deploy to production') {
            steps {
                script {
                    if (ROLLBACK == 'SUCCESS'){
                        sshagent(credentials : ['DigitalOcean_Servers']) {
                            sh 'exit 1'
                        }
                    } else {
                        try {
                            REQUESTED_ACTION = input(
                                id: 'Proceed', message: 'Deploy to production?')
                        } catch(err) { // input false
                            def user = err.getCauses()[0].getUser()
                            echo "Aborted by: [${user}]"
                        }
                    }
                }
            }
        }
        stage('Tag and Push Productive Image') {
            steps {
                sshagent(credentials : ['DigitalOcean_Servers']) {
                    sh 'ssh -o StrictHostKeyChecking=no root@<domain> uptime'
                    sh 'ssh root@<domain> "/opt/Docker/Sistema_Pieenso/push_image_productivo.sh"'
                }
            }
        }
        stage('Create Productive Container') {
            steps {
                sshagent(credentials : ['DigitalOcean_Servers']) {
                    sh 'ssh -o StrictHostKeyChecking=no root@<domain> uptime'
                    sh 'ssh root@<domain> "/opt/Docker/Sistema_Pieenso/create_container_productivo.sh -i <domain-harbor>:80/sistema_pieenso/sistema-pieenso:productivo -c sistema-pieenso"'
                }
            }
        }
    }
}