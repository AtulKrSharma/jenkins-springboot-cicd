def registry = 'https://trialwug6k0.jfrog.io'
pipeline {
    agent any
    tools{
        maven 'maven'
    }
    stages {
        stage('Checkout From Git') {
            steps {
                git branch:'main' , url: "https://github.com/AtulKrSharma/jenkins-springboot-cicd.git"
            }
        }
        stage ('Maven Parallel Stages') {
            parallel {
            stage ('Maven Validate'){
            steps {
                sh 'mvn validate'
            }
        }
        stage ('Maven Compile'){
            steps {
                sh 'mvn compile'
            }
        }
        
        stage ('Maven Package'){
            steps {
                sh 'mvn package'
            }
          }
         }
       }
    // stage('Sonar Analysis') {
    // steps {
    //     script {
    //         def scannerHome = tool 'sonar-scanner'
    //         withSonarQubeEnv('sonarserver') {
    //             sh """
    //             ${scannerHome}/bin/sonar-scanner \
    //             -Dsonar.organization=bkrrajmali \
    //             -Dsonar.projectName=SpringBootPet \
    //             -Dsonar.projectKey=bkrrajmali_springbootpet \
    //             -Dsonar.java.binaries=target
    //             """
    //                     }
    //                 }
    //             }
    //         }
    // stage("Quality Gate") {
    //         steps {
    //           timeout(time: 1, unit: 'MINUTES') {
    //             waitForQualityGate abortPipeline: true, credentialsId: 'sonar'
    //           }
    //         }
    //       }
        stage("Jar Publish") {
            steps {
                script {
                        echo '<--------------- Jar Publish Started --------------->'
                         def server = Artifactory.newServer url:registry+"/artifactory" ,  credentialsId:"jfrogaccess"
                         def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}";
                         def uploadSpec = """{
                              "files": [
                                {
                                  "pattern": "target/petclinic.war",
                                  "target": "demomaven-libs-release",
                                  "flat": "false",
                                  "props" : "${properties}",
                                  "exclusions": [ "*.sha1", "*.md5"]
                                }
                             ]
                         }"""
                         def buildInfo = server.upload(uploadSpec)
                         buildInfo.env.collect()
                         server.publishBuildInfo(buildInfo)
                         echo '<--------------- Jar Publish Ended --------------->'  
                
                }
            }   
        } 
        stage("Build Docker Image and TAG") {
            steps {
              script {
                sh 'docker build -t springboot:latest .'
              }
            }
        }
        stage("Trivy Scan") {
            steps {
              script {
                sh 'trivy image --format table --scanners vuln -o trivy-image-report.html springboot:latest'
              }
            }
        }
        stage("Push Docker Image to AWS ECR") {
            steps {
              script {
                sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 175157388210.dkr.ecr.us-east-1.amazonaws.com'
                sh 'docker tag springboot:latest 175157388210.dkr.ecr.us-east-1.amazonaws.com/myrepo:latest'
                sh 'docker push 175157388210.dkr.ecr.us-east-1.amazonaws.com/myrepo:latest'
              }
            }
        }
          stage("Deploy To Kubernetes") {
            steps {
              script {
                sh 'aws eks update-kubeconfig --region us-east-1 --name eksdemo2'
                sh 'kubectl apply -f k8s/sprinboot-deployment.yaml'
              }
            }
        }
      }
    }