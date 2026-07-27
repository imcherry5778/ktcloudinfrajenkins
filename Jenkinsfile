pipeline {
  agent any
  stages {
    stage('git scm update') {
      steps {
        git url: 'https://github.com/imcherry5778/ktcloudinfrajenkins.git', branch: 'main'
      }
    }
    stage('docker build and push image') {
      steps {
        withCredentials([
          usernamePassword(
            credentialsId: 'dockerhub-ci-pat',
            usernameVariable: 'DOCKERHUB_USER',
            passwordVariable: 'DOCKERHUB_PAT'
          )
        ]) {
          sh '''
            set -eu

            set +x
            printf '%s\n' "$DOCKERHUB_PAT" |
              docker login --username "$DOCKERHUB_USER" --password-stdin
            set -x

            docker build -t imcherry5778/ktcloudinfra4:0727 .
            docker push imcherry5778/ktcloudinfra4:0727
          '''
        }
      }
      post {
        always {
          sh 'docker logout >/dev/null 2>&1 || true'
        }
      }
    }
    stage('delivery and deployment using k8s') {
      steps {
        sh '''
          set -eu

          ansible master \
            -m ansible.builtin.copy \
            -a "src=${WORKSPACE}/deploy.yml dest=./deploy.yml owner=root group=root mode=0644"

          ansible master \
            -m ansible.builtin.shell \
            -a "kubectl apply -f ./deploy.yml"
        '''
      }
    }
  }
}
