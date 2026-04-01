pipeline{
    agent any
    environment{
        ANSIBLE_SERVER = "172.105.52.24 "
    }
    stages{
        stage("copy files to ansible server"){
            steps {
                script{
                    echo "Copy all necessary file to ansible control node"
                    sshagent(["ansible-server-key"]){
                        
                        // copy ansible files to ansible-server
                        sh "scp -o StrictHostKeyChecking=no ansible/* root@${ANSIBLE_SERVER}:/root"
                        
                        withCredentials([
                            sshUserPrivateKey(
                                credentialsId: "ec2-server-key", 
                                keyFileVariable: "keyfile",     // to create file of credentials, not user content directly
                                usernameVariable: "user"
                            )
                        ]){ 
                            // copy ec2 credentials (ssh private key to connect to ec2 machines) to ansible server
                            sh 'scp $keyfile root@$ANSIBLE_SERVER:/root/ssh-key.pem'
                        }
                    }
                }
            }
        }
        stage("Execute ansible playbook"){
            steps{
                script{
                    echo "Calling ansible playbook to configure ec2 instances"

                    def remote = [:]
                    remote.name = "ansible-server"
                    remote.host = ANSIBLE_SERVER
                    remote.allowAnyHosts = true
                    
                    withCredentials([
                        sshUserPrivateKey(
                            credentialsId: "ansible-server-key", 
                            keyFileVariable: "keyfile",     // to create file of credentials, not user content directly
                            usernameVariable: "user"
                        )
                    ]){ 
                        remote.user = user
                        remote.identityFile = keyfile

                        // execute command
                        sshCommand remote: remote, command: "ansible-playbook deploy-docker.yaml"
                    }
                }
            }
        }
    }
}