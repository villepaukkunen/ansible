pipeline {
    agent {
        kubernetes {
            yaml '''
            apiVersion: v1
            kind: Pod
            spec:
              containers:
                - name: ansible
                  image: docker.io/alpine/ansible
                  command:
                    - cat
                  tty: true
                  securityContext:
                    privileged: true
            '''
        }
    }
    environment {
        SSH_CREDS = credentials('ansible')
        VAULT_KEY = credentials('vault-key')
    }
    triggers {
        pollSCM 'H/5 * * * *'
    }
    stages {
        stage('Set environment variables') {
            steps {
                script{
                    env.LIMIT = "server"
                }
            }
        }
        stage('Configure servers') {
            steps {
                container('ansible') {
                    sh '''
                    ansible-playbook local.yml --private-key $SSH_CREDS --ssh-extra-args "-o StrictHostKeyChecking=no" --vault-password-file $VAULT_KEY --limit $LIMIT -u $SSH_CREDS_USR
                    '''
                }
            }
        }
    }
}