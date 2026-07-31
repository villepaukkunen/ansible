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
                    privileged: false
            '''
        }
    }
    environment {
        SSH_CREDS = credentials('ssh-key')
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
                    env.USER = "ansible"
                }
            }
        }
        stage('Create vault key file') {
            steps {
                sh '''
                echo $VAULT_KEY > .vault_key
                '''
            }
        }
        stage('Configure servers') {
            steps {
                container('ansible') {
                    sh '''
                    ansible-playbook local.yml --private-key $SSH_CREDS --ssh-extra-args "-o StrictHostKeyChecking=no" --vault-password-file .vault_key --limit $LIMIT -u $USER
                    '''
                }
            }
        }
    }
}