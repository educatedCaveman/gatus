pipeline {
    agent any 

    environment {
        ANSIBLE_REPO = '/var/lib/jenkins/workspace/ansible_master'
        WEBHOOK = credentials('JENKINS_DISCORD')
        PORTAINER_PI_WEBHOOK = credentials('PORTAINER_WEBHOOK_PI_GATUS')
    }

    //triggering periodically so the code is always present
    // run every friday at 3AM
    triggers { cron('0 3 * * 5') }

    stages {
        // deploy code to lv-426.lab, when the branch is 'dev_test'
        stage('deploy code') {
            steps {
                // deploy configs to DEV
                echo 'deploy docker config files (DEV)'
                sh 'ansible-playbook -i ${ANSIBLE_REPO}/hosts.ini ${ANSIBLE_REPO}/deploy/docker/deploy_docker_compose_pi.yml -e repo="gatus"'
                echo 'decrypt repo'
                sh 'ansible-playbook -i ${ANSIBLE_REPO}/hosts.ini -l "dilithium" ${ANSIBLE_REPO}/deploy/git-crypt.yml -e repo="gatus"'
            }
        }
        // trigger portainer redeploy
        // separated out so this only gets run if the ansible playbook doesn't fail
        stage('redeploy portainer stack') {
            when { branch 'dev_test' }
            steps {
                // deploy configs to DEV
                echo 'Redeploy DEV stack'
                sh 'http post ${PORTAINER_PI_WEBHOOK}'
            }
        }

    }
    post {
        always {
            discordSend \
                description: "${JOB_NAME} - build #${BUILD_NUMBER}", \
                // footer: "Footer Text", \
                // link: env.BUILD_URL, \
                result: currentBuild.currentResult, \
                // title: JOB_NAME, \
                webhookURL: "${WEBHOOK}"
        }
    }
}

