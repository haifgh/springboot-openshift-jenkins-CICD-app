pipeline {
  agent {
      label 'maven'
  }
   
 triggers {
       pollSCM('*/5 * * * *')
    }
  
 options { disableConcurrentBuilds() }
  
  stages {     
    stage('Test App') {
      steps {
        bat "mvn clean test"
      }
    }
    stage('Quality Check :: Sonarqube & JaCoCo') {
      steps {
        bat  "mvn sonar:sonar"
      }}
    stage('Insatll App') {
      steps {
        bat  "mvn install"
      }}
      stage("Deploiement dans nexus ") {
     		 steps{
              // If you are using Windows then you should use "bat" step
              // Since unit testing is out of the scope we skip them
      	bat "mvn deploy:deploy-file -DgroupId=com.example -DartifactId=spring-example -Dversion=0.0.1-SNAPSHOT -DgeneratePom=true -Dpackaging=jar -DrepositoryId=deploymentRepo -Durl=http://localhost:8081/repository/maven-releases/ -Dfile=Jenkins/workspace/mayTest/targetspring-example-0.0.1-SNAPSHOT.jar"
                }
            }
	stage('Email Notifications'){
                 steps{
                 mail bcc: '', body: '''Hello , 
                A Build has been executed on Your Project Timesheet , if you notice any bugs or abnormal behaviour please contact your team leader
                Best Regards , 
                Ghabri Haifa''', 
                cc: '', from: '', replyTo: '', subject: 'A Build was executed on timesheet', to: 'haifa.ghabri@esprit.tn'
             
                 }
                 } 
        	
    stage('Create Image Builder') {
      when {
        expression {
          openshift.withCluster() {
            return !openshift.selector("bc", "springbootapp").exists();
          }
        }
      }
      steps {
        script {
          openshift.withCluster() {
            openshift.newBuild("--name=springbootapp","--image-stream=openjdk18-openshift:1.1", "--binary")
          }
        }
      }
    }
    stage('Build Image') {
      steps {
        script {
          openshift.withCluster() {
            openshift.selector("bc", "springbootapp").startBuild("--from-file=target/spring-example-0.0.1-SNAPSHOT.jar", "--wait")
          }
        }
      }
    }
    stage('Promote to UAT') {
      steps {
        script {
          openshift.withCluster() {
            openshift.tag("springbootapp:latest", "springbootapp:uat")
          }
        }
      }
    }
    stage('Create UAT') {
      when {
        expression {
          openshift.withCluster() {
            return !openshift.selector('dc', 'springbootapp-uat').exists()
          }
        }
      }
      steps {
        script {
          openshift.withCluster() {
            openshift.newApp("springbootapp:latest", "--name=springbootapp-uat").narrow('svc').expose()
          }
        }
      }
    }
    stage('Promote PROD') {
      steps {
        script {
          openshift.withCluster() {
            openshift.tag("springbootapp:uat", "springbootapp:prod")
          }
        }
      }
    }
    stage('Create PROD') {
      when {
        expression {
          openshift.withCluster() {
            return !openshift.selector('dc', 'springbootapp-prod').exists()
          }
        }
      }
      steps {
        script {
          openshift.withCluster() {
            openshift.newApp("springbootapp:prod", "--name=springbootapp-prod").narrow('svc').expose()
          }
        }
      }
    }
  }
}
