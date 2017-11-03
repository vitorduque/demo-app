podTemplate(label: 'jenkins-pipeline',
    containers: [
        containerTemplate(
            name: 'node-karma-chrome',
            image: 'hellohq/node-karma-chrome:latest',
            ttyEnabled: true,
            command: 'cat'
        ),
        containerTemplate(
            name: 'jnlp',
            image: 'jenkins/jnlp-slave:3.10-1-alpine',
            args: '${computer.jnlpmac} ${computer.name}',
            workingDir: '/home/jenkins'
        ),
        containerTemplate(
            name: 'docker',
            image: 'docker:latest',
            ttyEnabled: true,
            command: 'cat'
        ),
        containerTemplate(
            name: 'envsubst',
            image: 'acm1/gettext:latest',
            ttyEnabled: true,
            command: 'cat'
        ),
        containerTemplate(
            name: 'kubectl',
            image: 'lachlanevenson/k8s-kubectl:latest',
            command: 'cat',
            ttyEnabled: true
        )
    ],
    volumes:[
      hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock'),
    ]
  )

{
  node('jenkins-pipeline'){

    def imageName = 'vtrduque/demo-app'
    def prefix = env.BRANCH_NAME
    def version
    def fullName

    stage('Checkout'){
      scmVars = checkout scm
      version = "${scmVars.GIT_COMMIT}"
    }

    stage('NPM'){
      container('node-karma-chrome'){
          stage('build') {
            sh 'npm install'
          }
      }
    }

    stage('Build container'){
      container('docker'){
        stage('Build'){
          sh "docker build -t ${imageName}  ."
        }

        stage('Push to registry'){

          docker.withRegistry('https://registry.hub.docker.com', 'docker-hub') {
            fullName = "$prefix-${scmVars.GIT_COMMIT}"
            docker.image(imageName).push(fullName)
          }
        }
      }
    }

    stage('Prepare deployment'){
      container('envsubst'){
        if (env.BRANCH_NAME == 'dev') {
          sh "sh deploy.sh $fullName develop"
          sh "cat .generated/deployment.yml"
        }else if (env.BRANCH_NAME == 'master') {
          sh "sh deploy.sh $fullName prod"
          sh "cat .generated/deployment.yml"
        }
      }
    }

    stage('Deploy to k8s cluster'){
      container('kubectl'){
        if (env.BRANCH_NAME == 'dev' || env.BRANCH_NAME == 'master') {
          sh "kubectl apply -f .generated/deployment.yml"
        }
      }
    }

  }
}
