pipeline {
  agent any
  tools {
  
  maven 'Maven'
   
  }
    stages {

      stage ('Checkout SCM'){
        steps {
          checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'git', url: 'https://github.com/adegokeobafemi/july-set.git']]])
        }
      }
	  
	  stage ('Build')  {
	      steps {
            dir('webapp'){
            sh "pwd"
            sh "ls -lah"
            sh "mvn package"
          }
        }
         
      }
   
     stage ('SonarQube Analysis') {
        steps {
              withSonarQubeEnv('sonar') {
                
				dir('webapp'){
                 sh 'mvn -U clean install sonar:sonar'
                }
				
              }
            }
      }

    stage ('Artifactory configuration') {
            steps {
                rtServer (
                    id: "jfrog",
                    url: "http://52.12.83.89:8082/artifactory",
                    credentialsId: "jfrog"
                )

                rtMavenDeployer (
                    id: "MAVEN_DEPLOYER",
                    serverId: "jfrog",
                    releaseRepo: "joint-devops-libs-release-local",
                    snapshotRepo: "joint-devops-libs-snapshot-local"
                )

                rtMavenResolver (
                    id: "MAVEN_RESOLVER",
                    serverId: "jfrog",
                    releaseRepo: "joint-devops-libs-release",
                    snapshotRepo: "joint-devops-libs-snapshot"
                )
            }
    }

    stage ('Deploy Artifacts') {
            steps {
                rtMavenRun (
                    tool: "Maven", // Tool name from Jenkins configuration
                    pom: 'webapp/pom.xml',
                    goals: 'clean install',
                    deployerId: "MAVEN_DEPLOYER",
                    resolverId: "MAVEN_RESOLVER"
                )
         }
    }

    stage ('Publish build info') {
            steps {
                rtPublishBuildInfo (
                    serverId: "jfrog"
             )
        }
    }

    stage('Copy Dockerfile & Playbook to Ansible Server') {
            
            steps {
                  sshagent(['ssh_agent']) {
                       sh "chmod 400  oregon-kp.pem"
                       sh "ls -lah"
                        sh "scp -i oregon-kp.pem -o StrictHostKeyChecking=no Dockerfile ubuntu@54.68.224.137:/home/ubuntu"
                        sh "scp -i oregon-kp.pem -o StrictHostKeyChecking=no Dockerfile ubuntu@54.68.224.137:/home/ubuntu"
                        sh "scp -i oregon-kp.pem -o StrictHostKeyChecking=no dockerhub.yaml ubuntu@54.68.224.137:/home/ubuntu"
                    }
                }
        } 

    stage('Build Container Image') {
            
            steps {
                  sshagent(['ssh_key']) {
                        sh "ssh -i oregon-kp.pem -o StrictHostKeyChecking=no ubuntu@54.68.224.137 -C \"ansible-playbook  -vvv -e build_number=${BUILD_NUMBER} dockerhub.yaml\""       
                    }
                }
        } 

    stage('Copy Deployment & Service Defination to K8s Master') {
            
            steps {
                  sshagent(['ssh_key']) {
                        sh "scp -i oregon-kp.pem -o StrictHostKeyChecking=no deployment.yaml ubuntu@44.233.10.40:/home/ubuntu"
                        sh "scp -i oregon-kp.pem -o StrictHostKeyChecking=no service.yaml ubuntu@44.233.10.40:/home/ubuntu"
                    }
                }
        } 

    stage('Waiting for Approvals') {
            
        steps{

				input('Test Completed ? Please provide  Approvals for Prod Release ?')
			 }
    }     
    stage('Deploy Artifacts to Production') {
            
            steps {
                  sshagent(['ssh_key']) {
                        //sh "ssh -i oregon-kp.pem -o StrictHostKeyChecking=no ubuntu@44.233.10.40 -C \"kubectl set image deployment/ranty customcontainer=adegokeobafemi/july-devops:${BUILD_NUMBER}\""
                        sh "ssh -i oregon-kp.pem -o StrictHostKeyChecking=no ubuntu@44.233.10.40 -C \"kubectl delete deployment ranty && kubectl delete service ranty\""
                        sh "ssh -i oregon-kp.pem -o StrictHostKeyChecking=no ubuntu@44.233.10.40 -C \"kubectl apply -f deployment.yaml\""
                        sh "ssh -i oregon-kp.pem -o StrictHostKeyChecking=no ubuntu@44.233.10.40 -C \"kubectl apply -f service.yaml\""
                    }
                }  
        } 
   } 
}


