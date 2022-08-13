def BUILDER = "jenkins-agent-gke"
def project = "submission-adi-purnomo"
def appname = "hello-world"

pipeline {
  agent {
    kubernetes {
      // Without cloud, Jenkins will pick the first cloud in the list
      cloud "metaverse-cluster"
      label "jenkins-agent"
      yamlFile "jenkins-build-pod.yaml"
    }
  }

  stages {
    stage("Build") {
      environment {
        IMAGE_REPO = "asia.gcr.io/${project}/${appname}"
      } 
      steps {
        dir("hello-app") {
          withCredentials([file(credentialsId: "${BUILDER}" , variable: "builder")]) {
            container("gcloud") {
              // Cheat by using Cloud Build to help us build our container
              writeFile file: 'key.json', text: readFile(builder)
              sh "gcloud auth activate-service-account --key-file=key.json"
              // sh "gcloud builds submit --project ${project} --tag ${IMAGE_REPO}:${IMAGE_TAG} ."
              sh "gcloud builds submit --project ${project} -t ${IMAGE_REPO}:${GIT_COMMIT}"
            }
          }
        }
      }
    }

    stage("Deploy") {
      steps {
        container("kubectl") {
          sh "kubectl get po -A"
        }
      }
    }
  }
}
