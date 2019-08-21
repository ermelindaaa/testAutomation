def remote = [:]
remote.name = "ubuntu"
remote.host = ""
remote.allowAnyHosts = true
node {
   withCredentials(
        [[
            $class: 'AmazonWebServicesCredentialsBinding',
            accessKeyVariable: 'AWS_ACCESS_KEY_ID',
            credentialsId: '39c07877-ebc4-4f70-a4ca-084feda446e1',  // ID of credentials in kubernetes
            secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
        ]]) {
       stage("create EC2 instance"){
            ID = sh (script: 'aws ec2 run-instances --image-id ami-0085d4f8878cddc81 --count 1 --instance-type t2.micro --key-name Taleas456 --security-group-ids sg-88d34feb --subnet-id subnet-166e626b --region eu-central-1 --query \'Instances[0].InstanceId\'',returnStdout: true)
        }
        stage("get the EC2 external ip"){
            remote.host = sh (script: "aws ec2 describe-instances --query \'Reservations[0].Instances[0].PublicIpAddress\' --instance-ids $ID",returnStdout: true)
        }
    }
   withCredentials([sshUserPrivateKey(credentialsId: '01eb9d49-682c-4e68-94a6-ec77889de9aa', keyFileVariable: 'identity', passphraseVariable: '', usernameVariable: 'userName')]) {
      remote.user = userName
      remote.identityFile = identity
      stage("install awscli") {
            sh 'sudo apt-get update'
            sh 'sudo apt-get install awscli -y'
      }
      stage("install kops"){
            sh 'curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d \'\"\' -f 4)/kops-linux-amd64 ' 
            sh 'chmod +x kops-linux-amd64'
            sh 'sudo mv kops-linux-amd64 /usr/local/bin/kops'
      }
      stage("install kubectl"){
            sh 'curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl'
            sh 'chmod +x ./kubectl'
            sh 'sudo mv ./kubectl /usr/local/bin/kubectl'
      }
       withCredentials(
        [[
            $class: 'AmazonWebServicesCredentialsBinding',
            accessKeyVariable: 'AWS_ACCESS_KEY_ID',
            credentialsId: '39c07877-ebc4-4f70-a4ca-084feda446e1',  // ID of credentials in kubernetes
            secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
        ]]) {
            stage("create s3 bucket"){
                sh 'aws configure set region eu-central-1'
                sh 'aws s3 mb s3://k8s.taleas.in'
            }
           stage("Expose environment variable"){
                sh 'export KOPS_STATE_STORE=s3://k8s.taleas.in'
           }
            stage("generate ssh-keygen"){
                sh 'sudo chmod -R 700 /root/.ssh/id_rsa'
                sh 'sudo ssh-keygen -t rsa -N "" -f /root/.ssh/id_rsa -y'
            }
            stage("create cluster configurations"){
               sh 'sudo chmod -R 777 /root/'
                sh 'sudo chmod -R 777 /root/.ssh/'
                sh 'kops create cluster k8s.tea.in --zones eu-central-1b --node-size t2.micro --master-size t2.micro --node-count 2 --master-zones eu-central-1b --ssh-public-key /root/.ssh/id_rsa.pub --state s3://k8s.taleas.in --dns-zone Z1EDG0NQOJK25F --yes'
                sh 'sudo chmod -R 700 /root/.ssh/'
                sh 'sudo chmod -R 700 /root/'
            }
            stage("create the cluster"){
                sh 'kops update cluster k8s.tea.in --state s3://k8s.taleas.in --yes'
            }
           stage("wait")
          { //echo "Waiting for EBS snapshot"
             //sh 'aws ec2 wait snapshot-completed --snapshot-ids snap-aabbccdd'
             //echo "EBS snapshot completed"
             out.println(System.currentTimeMillis())
            Thread.sleep(10000000)
            out.println(System.currentTimeMillis())
          }
            stage("create the pod"){
                sh 'kubectl run my-app --image=grisildarr/repository:firsttry --port=8080'
            }
            stage("expose the pod and run the container"){
                sh 'kubectl expose deployment my-app --type=LoadBalancer --port=8080 --target-port=8080'
            }
            stage("activate the autoscaling"){
                sh 'kubectl autoscale deployment test cpu-percent=70 --min=2 --max=5'
            }
             
        }
    }
}
