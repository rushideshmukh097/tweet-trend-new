def registry = 'https://rushi97.jfrog.io'
def imageName = 'rushi97.jfrog.io/valaxy-docker-local/ttrend'
def version   = '2.0.2'

pipeline {
    agent {
        node {
            label 'maven'
        }
    }

    environment {
        PATH = "/opt/apache-maven-3.9.3/bin:$PATH"
    }
    
    stages {
        stage("build") {
            steps {
               sh 'mvn clean deploy'
            }
        }

        stage("Jar Publish") {
            steps {
                script {
                    echo '<--------------- Jar Publish Started --------------->'
                     def server = Artifactory.newServer url:registry+"/artifactory" ,  credentialsId:"artifact-cred"
                     def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}";
                     def uploadSpec = """{
                          "files": [
                            {
                              "pattern": "jarstaging/(*)",
                              "target": "maven-libs-release-local/{1}",
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

         
        stage(" Docker Build ") {
           steps {
                script {
                 echo '<--------------- Docker Build Started --------------->'
                 app = docker.build(imageName+":"+version)
                 echo '<--------------- Docker Build Ends --------------->'
                }
            }
        }

        stage (" Docker Image publish to Jfrog  "){
            steps {
                script {
                    echo '<--------------- Docker Publish Started --------------->'  
                    docker.withRegistry(registry, 'artifact-cred'){
                      app.push()
                    }    
                   echo '<--------------- Docker Publish Ended --------------->'  
                }
            }
        }
        stage ("App deploy by Kubernetes") {
            steps {
                script {
                    sh './deploy.sh'
                }
            }
        }
    }    
}


     
         
