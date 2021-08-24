pipeline {
  agent any
  tools {
  
  maven 'mymaven'
   
  }
    stages {

      stage ('Checkout SCM'){
        steps {
            git branch: 'main', url: 'https://github.com/vtejapy/iwayq-proj.git'
        }
      }
          
          stage ('Build')  {
              steps {
          
            dir('java-source'){
            sh "mvn package"
          }
        }
         
      }
   
     stage ('SonarQube Analysis') {
        steps {
              withSonarQubeEnv('mysonar') {
                
                                dir('java-source'){
                 sh 'mvn -U clean install sonar:sonar'
                }

              }
            }
      }

    stage ('Artifactory configuration') {
            steps {
                rtServer (
                    id: "myjfrog",
                    url: "http://10.5.5.12:8082/artifactory",
                    credentialsId: "myjfrog"
                )

                rtMavenDeployer (
                    id: "MAVEN_DEPLOYER",
                    serverId: "myjfrog",
                    releaseRepo: "iq-libs-release",
                    snapshotRepo: "iq-libs-snapshot"
                )

                rtMavenResolver (
                    id: "MAVEN_RESOLVER",
                    serverId: "myjfrog",
                    releaseRepo: "iq-libs-release",
                    snapshotRepo: "iq-libs-snapshot"
                )
            }
    }

    stage ('Deploy Artifacts') {
            steps {
                rtMavenRun (
                    tool: "mymaven", // Tool name from Jenkins configuration
                    pom: 'java-source/pom.xml',
                    goals: 'clean install',
                    deployerId: "MAVEN_DEPLOYER",
                    resolverId: "MAVEN_RESOLVER"
                )
         }
    }

    stage ('Publish build info') {
            steps {
                rtPublishBuildInfo (
                    serverId: "myjfrog"
             )
        }
    }

    stage('Copy Dockerfile & Playbook to Ansible Server') {
            
            steps {
                  sshagent(['node4']) {
                       
                        sh 'scp -o StrictHostKeyChecking=no Dockerfile vagrant@10.5.5.13:/home/vagrant'
                        sh 'scp -o StrictHostKeyChecking=no create-container-image.yaml vagrant@10.5.5.13:/home/vagrant'
                    }
                }
            
        } 
    stage('Build Container Image') {
            
            steps {
                  sshagent(['node4']) {
                       
                        sh 'ssh -o StrictHostKeyChecking=no vagrant@10.5.5.13 -C \"sudo ansible-playbook create-container-image.yaml\"'
                        
                    }
                }
            
        } 
    

     
    stage('Deploy Artifacts to Production') {
            
            steps {
                  sshagent(['node4']) {
                       
                        sh 'ssh -o StrictHostKeyChecking=no  vagrant@10.5.5.13 -C \"sudo docker run -d -p 8080:8008 tomcat"'
                       
                        
                    }
                }
            
        } 
         
   } 
}
