pipeline {
  agent {
    kubernetes {
      yaml """
kind: Pod
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    imagePullPolicy: Always
    command:
    - /busybox/cat
    tty: true
    volumeMounts:
      - name: docker-config
        mountPath: /kaniko/.docker
  volumes:
    - name: docker-config
      projected:
        sources:
        configMap:
          name: docker-config
"""
    }
  }
  stages {
        stage('Approval') {
      when {
        branch 'main'
      }
      steps {
        script {
          def plan = 'frontend CI'
          input message: "Do you want to build and push?",
              parameters: [text(name: 'Plan', description: 'Please review the work', defaultValue: plan)]
        }
      } 
    }

    stage('Build with Kaniko') {
      steps {
        container(name: 'kaniko', shell: '/busybox/sh') {
          
          withCredentials([
          usernamePassword
            (credentialsId: 'github', 
             usernameVariable: 'USERNAME', 
             passwordVariable: 'GIT_TOKEN'
            )
          ])

          {
            sh '''#!/busybox/sh
            /kaniko/executor \\
            --git branch=main \\
            --context=git://$USERNAME:$GIT_TOKEN@github.com/namhyunCHO/eshop-MSA.git \\
            --context-sub-path=eshop-currencyservice \\
            --destination=450211143449.dkr.ecr.us-east-1.amazonaws.com/eshop-currencyservice:latest
            '''
          }
        }
      }
    }
  }  

  post {
    success { 
      slackSend(channel: '#3조-ci-notice', color: 'good', message: 'adservice CI success - from namhyunCHO')
    }
    failure {
      slackSend(channel: '#3조-ci-notice', color: 'danger', message: 'adservice CI fail - from namhyunCHO')
    }
  }

}
