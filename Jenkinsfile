podTemplate(
  cloud: 'openshift',
  yaml:'''
  spec:
    serviceAccount: jenkins-priviliged
    containers:
    - name: podman
      image: quay.io/podman/stable
      command: ['cat']
      tty: true
      alwaysPullImage: true
      securityContext:
        privileged: true
    - name: helm
      image: docker.io/alpine/helm
      command: ['cat']
      tty: true
      alwaysPullImage: true
      securityContext:
        privileged: true
    '''
  ) {
  node(POD_LABEL) {
    def http_proxy = '172.24.115.9:8080'
    def https_proxy  = '172.24.115.9:8080'
    // // Define build ID 
    // pritln 'Prepare date'
    // def now = new Date()
    // def buildDate = now.format("yyyyMMdd'T'HHmmss'Z'", TimeZone.getTimeZone('UTC'))
    // def BUILD=${builDate}_${BUILD_NUMBER}
    // echo 'Build date: ' + BUILD

    container('podman') {
      sh "echo $http_proxy"
      sh "echo Running tasks into $POD_CONTAINER"
      sh "echo Build number: $BUILD_NUMBER"
      stage('Clone Repo'){
        git branch: 'main', credentialsId: '45b047b8-ee96-47fb-97e8-75607736a728', url: 'https://github.kyndryl.net/alessandro-blasetti-CIC/Trenitalia_NewB2C.git'
      }
      stage('Build Image') {
        withEnv(
          [
            'http_proxy=172.24.115.9:8080',
            'https_proxy=172.24.115.9:8080'
          ]
        ){
          sh('podman build  -t test/newb2c SVIL_WEB_liberty_401A')
        }
      }
      stage("Push Image"){
        withCredentials([usernamePassword(credentialsId: 'podman-jenkins-robot', usernameVariable: 'CRED_USR', passwordVariable: 'CRED_PSW')]) {
          sh('podman login -u $CRED_USR -p $CRED_PSW image-registry.openshift-image-registry.svc:5000 --tls-verify=false')
          sh('podman tag test/newb2c image-registry.openshift-image-registry.svc:5000/test/newb2c:${BUILD_NUMBER}')
          sh('podman push image-registry.openshift-image-registry.svc:5000/test/newb2c:${BUILD_NUMBER} --tls-verify=false --creds $CRED_PSW:$CRED_PSW')
        }
      }
    }
    container('helm'){
      sh "echo Running tasks into $POD_CONTAINER"
      stage('Upgrade release'){
        git branch: 'main', credentialsId: '45b047b8-ee96-47fb-97e8-75607736a728', url: 'https://github.kyndryl.net/alessandro-blasetti-CIC/Trenitalia_NewB2C.git'
        sh "helm upgrade --install -f newb2c-helmchart/values.yaml  newb2c newb2c-helmchart -n test --set deployment.containers.image.tag=$BUILD_NUMBER"
        //sh """
        //  #!/bin/sh
        //  helm upgrade --install -f newb2c-helmchart/values.yaml  newb2c newb2c-helmchart -n test --set deployment.containers.image.tag=${BUILD_NUMBER}
        //"""
      }
    }
  }
}
