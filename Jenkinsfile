pipeline{
    agent any
    stages{
        stage("copy files to ansible server"){
            steps {
                script{
                    echo "Copy all necessary file to ansible control node"
                    sshagent(["ansible-server-key"]){
                        
                        // copy ansible files to ansible-server
                        sh "scp -o StrictHostKeyChecking=no ansible/* root@172.105.52.24:/root"
                        
                        withCredentials([
                            sshUserPrivateKey(
                                credentialsId: "ec2-server-key", 
                                keyFileVariable: "keyfile",     // to create file of credentials, not user content directly
                                usernameVariable: "user"
                            )
                        ]){ 
                            // copy ec2 credentials (ssh private key to connect to ec2 machines) to ansible server
                            sh 'scp $keyfile root@172.105.52.24:/root/ssh-key.pem'
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
                    remote.host = "172.105.52.24"
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