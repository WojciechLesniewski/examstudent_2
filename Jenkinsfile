node {
  def acr = 'acrdemo22.azurecr.io'
  def appName = 'whoamistudent2'
  def imageName = "${acr}/${appName}"
  def imageTag = "${imageName}:${env.BRANCH_NAME}.${env.BUILD_NUMBER}"
  def appRepo = "acrdemo22.azurecr.io/whoamistudent2:v1.0.0"

   checkout scm
 
 stage('Build the Image and Push to Azure Container Registry')
 {
   app = docker.build("${imageName}")
   withDockerRegistry([credentialsId: 'acr_auth', url: "https://${acr}"]) {
      app.push("${env.BRANCH_NAME}.${env.BUILD_NUMBER}")
                }
  }
 
 
 stage ("Deploy Application on Azure Kubernetes Service")
 {
  switch (env.BRANCH_NAME) {
    // Roll out to prod environment
    case "canary":
            // Create namespace 
        sh("sudo kubectl --kubeconfig ~wojcio/.kube/config get ns ${appName}-${env.BRANCH_NAME} || sudo kubectl --kubeconfig ~wojcio/.kube/config create ns ${appName}-${env.BRANCH_NAME}")
        withCredentials([usernamePassword(credentialsId: 'acr_auth', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
          sh "sudo kubectl --kubeconfig ~wojcio/.kube/config -n ${appName}-${env.BRANCH_NAME} get secret acr-auth || sudo kubectl --kubeconfig ~wojcio/.kube/config --namespace=${appName}-${env.BRANCH_NAME} create secret docker-registry acr-auth --docker-server ${acr} --docker-username $USERNAME --docker-password $PASSWORD"
        }
        sh("sed -i.bak 's#${appRepo}#${imageTag}#' ./k8s/canary/*.yaml")
        sh("sudo kubectl --kubeconfig ~wojcio/.kube/config --namespace=${appName}-${env.BRANCH_NAME} apply -f k8s/services/")
        sh("sudo kubectl --kubeconfig ~wojcio/.kube/config --namespace=${appName}-${env.BRANCH_NAME} apply -f k8s/canary/")
        sh("echo http://`kubectl --namespace=${appName} get service/${appName} --output=json | jq -r '.status.loadBalancer.ingress[0].ip'` > ${appName}")
        break

     //Release branch
     //Roll out to stage enviroment
     case "release":
        // Create namespace 
        sh("sudo kubectl --kubeconfig ~wojcio/.kube/config get ns ${appName}-${env.BRANCH_NAME} || sudo kubectl --kubeconfig ~wojcio/.kube/config create ns ${appName}-${env.BRANCH_NAME}")
        withCredentials([usernamePassword(credentialsId: 'acr_auth', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
          sh "sudo kubectl --kubeconfig ~wojcio/.kube/config -n ${appName}-${env.BRANCH_NAME} get secret acr-auth || sudo kubectl --kubeconfig ~wojcio/.kube/config --namespace=${appName}-${env.BRANCH_NAME} create secret docker-registry acr-auth --docker-server ${acr} --docker-username $USERNAME --docker-password $PASSWORD"
        }
        sh("sed -i.bak 's#${appRepo}#${imageTag}#' ./k8s/release/*.yaml")
        sh("sudo kubectl --kubeconfig ~wojcio/.kube/config --namespace=${appName}-${env.BRANCH_NAME} apply -f k8s/release/")
        sh("echo http://`kubectl --namespace=${appName} get service/${appName}-${env.BRANCH_NAME} --output=json | jq -r '.status.loadBalancer.ingress[0].ip'` > ${appName}")
        break
 
    // Roll out to production
    case "master":
        sh("sudo kubectl --kubeconfig ~wojcio/.kube/config get ns ${appName}-${env.BRANCH_NAME} || sudo kubectl --kubeconfig ~wojcio/.kube/config create ns ${appName}-${env.BRANCH_NAME}")
        withCredentials([usernamePassword(credentialsId: 'acr_auth', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
          sh "sudo kubectl --kubeconfig ~wojcio/.kube/config -n ${appName}-${env.BRANCH_NAME} get secret acr-auth || sudo kubectl --kubeconfig ~wojcio/.kube/config --namespace=${appName}-${env.BRANCH_NAME} create secret docker-registry acr-auth --docker-server ${acr} --docker-username $USERNAME --docker-password $PASSWORD"
        }
        sh("sed -i.bak 's#${appRepo}#${imageTag}#' ./k8s/production/*.yaml")
        sh("sudo kubectl --kubeconfig ~wojcio/.kube/config --namespace=${appName}-${env.BRANCH_NAME} apply -f k8s/services/")
        sh("sudo kubectl --kubeconfig ~wojcio/.kube/config --namespace=${appName}-${env.BRANCH_NAME} apply -f k8s/production/")
        sh("echo http://`kubectl --namespace=${appName}-${env.BRANCH_NAME} get service/${appName} --output=json | jq -r '.status.loadBalancer.ingress[0].ip'` > ${appName}")
        break
 
    // Roll out a dev environment
    default:
        // Create namespace if it doesn't exist
        sh("sudo kubectl --kubeconfig ~wojcio/.kube/config get ns ${appName}-${env.BRANCH_NAME} || kubectl create ns ${appName}-${env.BRANCH_NAME}")
        withCredentials([usernamePassword(credentialsId: 'acr_auth', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
          sh "sudo kubectl --kubeconfig ~wojcio/.kube/config -n ${appName}-${env.BRANCH_NAME} get secret acr-auth || kubectl --namespace=${appName}-${env.BRANCH_NAME} create secret docker-registry acr-auth --docker-server ${acr} --docker-username $USERNAME --docker-password $PASSWORD"
        }  
        sh("sed -i.bak 's#${appRepo}#${imageTag}#' ./k8s/dev/*.yaml")
        sh("sudo kubectl --kubeconfig ~wojcio/.kube/config --namespace=${appName}-${env.BRANCH_NAME} apply -f k8s/dev/")
        echo 'To access your environment run `kubectl proxy`'
        echo "Then access your service via http://localhost:8001/api/v1/namespaces/${appName}-${env.BRANCH_NAME}/services/${appName}:80/proxy/"    
    }
  }
}
