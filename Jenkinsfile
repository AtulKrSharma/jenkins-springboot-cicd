def registry = 'https://trialwug6k0.jfrog.io'
pipeline {
    agent any
    tools{
        maven 'maven'
    }

    environment {
        CLUSTER_NAME = "eks-atul-demo"  
        REGION = "us-east-1"
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
                        echo '<--------------- Jar Publish Started ---------------->'
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
            // Login to AWS ECR
            sh '''
                /usr/local/bin/aws ecr get-login-password --region us-east-1 \
                | docker login --username AWS --password-stdin 304182266883.dkr.ecr.us-east-1.amazonaws.com
            '''
            
            // Tag Docker image
            sh 'docker tag springboot:latest 304182266883.dkr.ecr.us-east-1.amazonaws.com/springboot:latest'
            
            // Push Docker image
            sh 'docker push 304182266883.dkr.ecr.us-east-1.amazonaws.com/springboot:latest'
        }
    }
}

// stage("Create EKS Cluster") {
//             steps {
//                 script {
//                     timeout(time: 30, unit: 'MINUTES') { // gives 30 minutes
//                         def clusterExists = sh(
//                             script: "aws eks describe-cluster --region ${REGION} --name ${CLUSTER_NAME} || echo 'not-found'",
//                             returnStdout: true
//                         ).trim()

//                         if (clusterExists.contains('not-found')) {
//                             echo "Cluster does not exist. Creating cluster..."
//                             sh """
//                             eksctl create cluster \\
//                                 --name ${CLUSTER_NAME} \\
//                                 --region ${REGION} \\
//                                 --nodes 2 \\
//                                 --node-type t3.medium
//                             """
//                         } else {
//                             echo "Cluster already exists. Skipping creation."
//                         }
//                     }
//                 }
//             }
//         }

        stage("Deploy To K8s") {
            steps {
                script {
                    sh "aws eks update-kubeconfig --region ${REGION} --name ${CLUSTER_NAME}"
                    sh 'kubectl apply -f k8s/springboot-deployment.yaml'
                }
            }
        }
    }
}
