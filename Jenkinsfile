def BUILDER = "jenkins-agent-gke"
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
        IMAGE_REPO = "asia.gcr.io/submission-adi-purnomo/hello-app/"
      } 
      steps {
        dir("hello-app") {
          withCredentials([file(credentialsId: "${BUILDER}" , variable: "builder")]) {
            container("gcloud") {
              // Cheat by using Cloud Build to help us build our container
              writeFile file: 'key.json', text: readFile(builder)
              sh "gcloud auth activate-service-account --key-file=key.json"
              // sh "gcloud builds submit --project ${project} --tag ${IMAGE_REPO}:${IMAGE_TAG} ."
              sh "gcloud builds submit -t ${IMAGE_REPO}:${GIT_COMMIT}"
            }
          }
        }
      }
    }

    stage("Deploy") {
      steps {
        container("kubectl") {
          sh """cat <<EOF | kubectl apply -f -
                apiVersion: apps/v1
                kind: Deployment
                metadata:
                  name: hello-app
                spec:
                  replicas: 2
                  selector:
                    matchLabels:
                      app: hello-app
                  template:
                    metadata:
                      labels:
                        app: hello-app
                    spec:
                      containers:
                      - name: hello-app
                        image: ${IMAGE_REPO}:${GIT_COMMIT}
                ---
                apiVersion: v1
                kind: Service
                metadata:
                  name: hello-app
                spec:
                  selector:
                    app: hello-app
                  ports:
                    - protocol: TCP
                      port: 80
                      targetPort: 8080
                """
          sh "kubectl rollout status deployments/hello-app"
        }
      }
    }
  }
}
