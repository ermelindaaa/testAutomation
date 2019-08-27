def remote = [:]
remote.name = "ubuntu"
def ID
def IP
remote.allowAnyHosts = true
node {
    
    
  withCredentials(
        [[
            $class: 'AmazonWebServicesCredentialsBinding',
            accessKeyVariable: 'AWS_ACCESS_KEY_ID',
            credentialsId: '39c07877-ebc4-4f70-a4ca-084feda446e1',  // ID of credentials in Jenkins
            secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
        ]]) 
    {
        stage("create EC2 instance")
        {
            ID = sh (script: 'aws ec2 run-instances --image-id ami-0085d4f8878cddc81 --count 1 --instance-type t2.micro --key-name Taleas456 --security-group-ids sg-88d34feb --subnet-id subnet-166e626b --region eu-central-1 --query \'Instances[0].InstanceId\'',returnStdout: true)
        }
        stage("get the EC2 external ip")
        {
            remote.host = sh (script: "aws ec2 describe-instances --query \'Reservations[0].Instances[0].PublicIpAddress\' --instance-ids $ID",returnStdout: true)
        }
    }


    
   withCredentials([sshUserPrivateKey(credentialsId: '01eb9d49-682c-4e68-94a6-ec77889de9aa', keyFileVariable: 'identity', passphraseVariable: '', usernameVariable: 'userName')]) 
    {
      remote.user = userName
      remote.identityFile = identity
      stage("Install Aws Cli") 
     {
            sh 'sudo apt-get update'
            sh 'sudo apt-get install awscli -y'
      }
      stage("Install Kops")
     {
            sh 'curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d \'\"\' -f 4)/kops-linux-amd64 ' 
            sh 'chmod +x kops-linux-amd64'
            sh 'sudo mv kops-linux-amd64 /usr/local/bin/kops'
      }
      stage("Install Kubectl")
     {
            sh 'curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl'
            sh 'chmod +x ./kubectl'
            sh 'sudo mv ./kubectl /usr/local/bin/kubectl'
      }
       withCredentials(
        [[
            $class: 'AmazonWebServicesCredentialsBinding',
            accessKeyVariable: 'AWS_ACCESS_KEY_ID',
            credentialsId:  '39c07877-ebc4-4f70-a4ca-084feda446e1',  
            secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
        ]]) 
        {
          stage("Create S3 bucket")
         {
               sh 'aws configure set region eu-central-1'
                sh 'aws s3 mb s3://k8s.taleas.in'
                
            }
          stage("Generate ssh-keygen")
          {
                sh 'sudo chmod -R 700 /root/.ssh/id_rsa'
                sh 'sudo ssh-keygen -t rsa -N "" -f /root/.ssh/id_rsa -y'
            }
          stage("Create cluster configurations")
          {
                sh 'sudo chmod -R 777 /root/'
                sh 'sudo chmod -R 777 /root/.ssh/'
                sh 'kops create cluster k8s.taleas.in --zones eu-central-1b --node-size t2.micro --master-size t2.micro --node-count 2 --master-zones eu-central-1b --master-volume-size 8 --node-volume-size 8 --ssh-public-key /root/.ssh/id_rsa.pub --state s3://k8s.taleas.in --dns-zone Z1NYSZ17QD2UGG --dns private --yes'
                sh 'sudo chmod -R 700 /root/.ssh/'
                sh 'sudo chmod -R 700 /root/'
            }
           stage("Create the cluser")
          {
                sh 'kops update cluster k8s.taleas.in --state s3://k8s.taleas.in --yes'
            }
            
           stage('Check Availability')
            {      waitUntil
                  {   try
                      {        
                            sh "kubectl get nodes"
                            return true
                        }
                       catch (Exception e)
                       {     return false
                        }
                    }
                }
            stage ("Deployment & Replicas")
            {
                sh 'kubectl run my-appTaleas --image=teaaa2000/repository:firsttry --replicas=2 --port=8080' 
            }
            stage ("Exposing the Deployment")
           {
                sh 'kubectl expose deployment my-appTaleas --type=LoadBalancer --port=8080 --target-port=8080'
            }
            stage ("Autoscaling")
           {
                sh 'kubectl autoscale deployment my-appTaleas --cpu-percent=50 --min=1 --max=10'
            }
            stage ("Update")
           {
                sh 'kubectl set image deployment/my-appTaleas my-appTaleas=grisildarr/repository:firsttry'
            }
        }
    }
}
