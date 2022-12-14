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
  openshift.withProject("loginform") {
  
    def buildConfigExists = openshift.selector("bc", "build-data-config").exists() 
    
    if(!buildConfigExists){ 
      openshift.newBuild("--name=build-data-config", "--docker-image=registry.redhat.io/jboss-eap-7/eap74-openjdk8-openshift-rhel7", "--binary") 
    } 
    
    openshift.selector("bc", "build-data-config").startBuild("--from-file=webapp/target/webapp.war", "--follow") } }

        }
      }
    }
    stage('Deploy') {
      steps {
        echo 'Deploying....'
        script {

          openshift.withCluster() { 
  openshift.withProject("loginform") { 
    def deployment = openshift.selector("dc", "build-data-config") 
    
    if(!deployment.exists()){ 
      openshift.newApp('build-data-config', "--as-deployment-config").narrow('svc').expose() 
    } 
    
    timeout(5) { 
      openshift.selector("dc", "build-data-config").related('pods').untilEach(1) { 
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
