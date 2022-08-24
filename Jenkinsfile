#! /usr/bin/env groovy

pipeline {

  agent {
    label 'maven'
  }

  stages {
    stage('Build') {
      steps {
        echo 'Building..'
        
        // Add steps here
      }
    }
    stage('Create Container Image') {
      steps {
        echo 'Create Container Image..'
        
        script {

          openshift.withCluster() { 
          openshift.withProject("ops-dev-dev") {
  
          def buildConfigExists = openshift.selector("bc", "openshift-jenkins").exists() 
    
          if(!buildConfigExists){ 
             openshift.newBuild("--name=openshift-jenkins", "--docker-image=registry.redhat.io/jboss-eap-7/eap74-openjdk8-openshift-rhel7", "--binary") 
      } 
    
          openshift.selector("bc", "openshift-jenkins").startBuild("--from-file=target/simple-servlet-0.0.1-SNAPSHOT.war", "--follow") } }

        }
      }
    }
    stage('Deploy') {
      steps {
        echo 'Deploying....'
        script {

          openshift.withCluster() { 
            openshift.withProject("ops-dev-dev") { 
            def deployment = openshift.selector("dc", "openshift-jenkins") 
    
            if(!deployment.exists()){ 
              openshift.newApp('openshift-jenkins', "--as-deployment-config").narrow('svc').expose() 
            } 
    
           timeout(5) { 
              openshift.selector("dc", "openshift-jenkins").related('pods').untilEach(1) { 
                return (it.object().status.phase == "Running") 
              } 
          } 
         } 
        }
        }
      }
    }
  }
}
