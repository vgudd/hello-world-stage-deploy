#!/usr/bin/env groovy

def label = "k8sagent-hello-world-stage-deploy"
def home = "/home/jenkins"
def workspace = "${home}/workspace/build-jenkins-operator"
def workdir = "${workspace}/src/github.com/jenkinsci/kubernetes-operator/"

podTemplate(label: label, yaml: """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: ubuntu
    image: ubuntu:16.04
    command: ['cat']
    tty: true
    env:
    - name: HTTPS_PROXY
      value: http://172.18.192.4:8080
    - name: https_proxy
      value: http://172.18.192.4:8080
    - name: NO_PROXY
      value: localhost,127.0.0.1,gw-bluedata.ncs.com
    - name: no_proxy
      value: localhost,127.0.0.1,gw-bluedata.ncs.com
"""
  ) {
    node(label) {
        stage('Checkout Workspace') {
           git url: 'https://github.hpe.com/hardik-dhi-parekh/hello-world-stage-deploy', branch: env.BRANCH_NAME
        }
        stage('YAML lint') {
          container('ubuntu') {
            sh 'apt-get update'
            sh 'apt-get install -y python3-pkg-resources'
            sh 'apt-get install -y yamllint'
            sh 'yamllint ./'
          }            
        }
        if (env.BRANCH_NAME == "master") {
        stage('Deploy') {
            container('ubuntu') {
             withCredentials([file(credentialsId: 'stage-kubeconfig', variable: 'KUBECONFIG')]) {
              sh "apt-get update"
              sh "apt-get install -y curl"
              sh "curl -Lo /usr/local/bin/kubectl https://storage.googleapis.com/kubernetes-release/release/v1.15.3/bin/linux/amd64/kubectl"
              sh "chmod +x /usr/local/bin/kubectl"
              def kubectl = "kubectl  --kubeconfig=\$KUBECONFIG"
              def namespace = sh (
                  script: 'kubectl  --kubeconfig=\$KUBECONFIG get ns | grep hello-world | awk \'{print $1}\'',
                  returnStdout: true
              )
              sh "echo ${namespace}"
              sh "${kubectl} get ns"
              if (namespace == "") {
                 sh "${kubectl} create ns hello-world"
              }
              sh "${kubectl} apply -f ./ -n hello-world"
              sh "sleep 5"
              sh "${kubectl} get svc -n hello-world -o yaml"
             }
            }
          }
        }
    }
}
