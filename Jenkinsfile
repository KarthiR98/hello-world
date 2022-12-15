pipeline {

  agent {
    label 'maven'
  }

  stages {
    stage('Build') {
      steps {
        echo 'Building..'
        sh 'mvn clean package'
      }
    }
    stage('Create Container Image') {
      steps {
        echo 'Create Container Image..'
        
        script {

          openshift.withCluster() { 
  openshift.withProject("supermarket") {
  
    def buildConfigExists = openshift.selector("bc", "gettingreadyconf").exists() 
    
    if(!buildConfigExists){ 
      openshift.newBuild("--name=gettingreadyconf", "--docker-image=registry.redhat.io/jboss-eap-7/eap74-openjdk8-openshift-rhel7", "--binary") 
    } 
    
    openshift.selector("bc", "gettingreadyconf").startBuild("--from-file=webapp/target/webapp.war", "--follow") } }

        }
      }
    }
    stage('Deploy') {
      steps {
        echo 'Deploying....'
        script {

          openshift.withCluster() { 
  openshift.withProject("supermarket") { 
    def deployment = openshift.selector("dc", "gettingreadyconf") 
    
    if(!deployment.exists()){ 
      openshift.newApp('gettingreadyconf', "--as-deployment-config").narrow('svc').expose() 
    } 
    
    timeout(5) { 
      openshift.selector("dc", "gettingreadyconf").related('pods').untilEach(1) { 
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
