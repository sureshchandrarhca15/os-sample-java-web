def OCP_INTERNAL_REGISTRY_URL = "default-route-openshift-image-registry.apps.cluster-dc92.sandbox775.opentlc.com"
def QUAY_REGISTRY_URL = "quay.apps.cluster-dc92.sandbox775.opentlc.com"
def QUAY_ORG_NAME = "redhat"
def QUAY_REPO_NAME = "ccapp"
def QUAY_IMAGE_TAG = env.BUILD_NUMBER

pipeline {

  environment { 
    APP_NAME='hello-world'
    DEV_NAMESPACE='hello-world-dev'
    IMAGE_BAKERY_NAMESPACE='image-bakery'
   }
  agent { label 'maven' }

  stages {

  stage('Build') {
    steps {
      script {
      try {
      
      sh "mvn clean install -Dmaven.test.skip=true"
      
        }
      catch (Exception e) {
      println "Failed to Maven Build - ${currentBuild.fullDisplayName}"
      throw e
      }
      }
    }
  }

  stage('Unit Test') {
    steps {
      script {  
      try {
      
      sh "mvn test -Dmaven.test.skip=true"
      
        }
      catch (Exception e) {
      println "Failed to Run Unit Test - ${currentBuild.fullDisplayName}"
      throw e
      }
      }
    }
  } 

  stage('Sonar Analysis') {
    steps {
      script {  
      try {
      withSonarQubeEnv('sonarqube_7_5') { 
        sh "mvn -e -B sonar:sonar -Dversion=${env.BUILD_NUMBER}"  
            }
      }
      catch (Exception e) {
      println "Failed to Run Sonar Analysis - ${currentBuild.fullDisplayName}"
      throw e
      }
      }
    }
  }

  stage('Quality Gate') {
    steps {
      script {  
      try {
      withSonarQubeEnv('sonarqube_7_5') {
            timeout(time: 1, unit: 'HOURS') {
              def qg = waitForQualityGate()
              if (qg.status != 'OK' && qg.status != 'WARN') {
                  error "Pipeline aborted due to quality gate failure: ${qg.status}"
              }
            }
          }
      }
      catch (Exception e) {
        println "Failed to Test Quality Gate  - ${currentBuild.fullDisplayName}"
        throw e
      }
     }
    }
  }
  stage('Create Image Builder') {
      when {
        expression {
          openshift.withCluster() {
            openshift.withProject(env.DEV_NAMESPACE) {
            return !openshift.selector("bc", "${env.APP_NAME}").exists();
            }
          }
        }
      }       
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject(env.DEV_NAMESPACE) {
              openshift.newBuild("--name=${env.APP_NAME}", "--image-stream=jboss-webserver50-tomcat9-openshift:1.0", "--binary=true")
            }
          }
        }
      }
    }

    stage('Build Image') {
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject(env.DEV_NAMESPACE) {
              openshift.selector("bc", "${env.APP_NAME}").startBuild("--from-file=target/ROOT.war","--follow","--wait=true")
            }
          }
        }
      }
    }

/*
    stage('Sign and Push Image to Quay') {
    agent { label 'image_sign' }
      environment
      {
        OCP_INTERNAL_REGISTRY_URL = "${OCP_INTERNAL_REGISTRY_URL}"
        OCP_NAMESPACE = "${env.IMAGE_BAKERY_NAMESPACE}"
        OCP_IMAGE_NAME = "${env.APP_NAME}"
        OCP_IMAGE_TAG = "latest"
        QUAY_REGISTRY_URL = "${QUAY_REGISTRY_URL}"
        QUAY_ORG_NAME = "${QUAY_ORG_NAME}"
        QUAY_REPO_NAME = "${QUAY_REPO_NAME}"
        QUAY_IMAGE_TAG = "${QUAY_IMAGE_TAG}"
        TERM = "xterm"
      }
      steps {
        withCredentials([string(credentialsId: 'gpg-key-id', variable: 'GPG_KEY_ID'), string(credentialsId: 'gpg_key_passphrase', variable: 'GPG_KEY_PASSPHRASE'), usernamePassword(credentialsId: 'kubeadmin-creds', passwordVariable: 'OCP_INTERNAL_REGISTRY_USER_TOKEN', usernameVariable: 'OCP_INTERNAL_REGISTRY_USERNAME'), usernamePassword(credentialsId: 'quay-registry-creds', passwordVariable: 'QUAY_REGISTRY_USER_PASSWORD', usernameVariable: 'QUAY_REGISTRY_USER_NAME')]) {
        script {
          openshift.withCluster() {
            openshift.withProject(env.DEV_NAMESPACE) { 
              sh """
                /root/kill_gpg_agent.sh 

		echo "Adding Signature to the Image, please wait"  
		/root/image_signature.exp
	      """
            }
          }
        }
      }
    }
    }
    stage('Tag Image to Dev') {
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject(env.DEV_NAMESPACE) {
              openshift.tag("${env.APP_NAME}:latest", "${env.APP_NAME}:dev")
          }
        }
      }
    }
  }

    stage('Deploy to Dev') {
      when {
        expression {
          openshift.withCluster() {
            openshift.withProject(env.DEV_NAMESPACE) {
            return !openshift.selector('dc', "${env.APP_NAME}").exists()
          }
        }
      }
    }
    steps {
      script {
        openshift.withCluster() {
          openshift.withProject(env.DEV_NAMESPACE) {
            openshift.newApp("${env.APP_NAME}:dev", "--name=${env.APP_NAME}").narrow('svc').expose()
            def dc = openshift.selector("dc", "${env.APP_NAME}")
            while (dc.object().spec.replicas != dc.object().status.availableReplicas) {
              sleep 10
            }
          }
        }
      }
    }
  }
*/

}
}
