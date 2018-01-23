#!/usr/bin/groovy

@Library('pipeline-lib') _

def pipeline = new io.vtrduque.Pipeline()

pipeline.helloWorld()

podTemplate(label: 'jenkins-pipeline',
    containers: [
        containerTemplate(name: 'node', image: 'node:latest', ttyEnabled: true, command: 'cat'),
        containerTemplate(name: 'docker', image: 'docker:latest', ttyEnabled: true, command: 'cat'),
        containerTemplate(name: 'envsubst', image: 'acm1/gettext:latest', ttyEnabled: true, command: 'cat'),
        containerTemplate(name: 'kubectl', image: 'lachlanevenson/k8s-kubectl:latest', ttyEnabled: true, command: 'cat')
    ],
    volumes:[hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock')]
)


{
  node('jenkins-pipeline'){

    def imageName = 'vtrduque/demo-app'
    def branch = env.BRANCH_NAME
    def version
    def fullName

    stage('Checkout'){

      scmVars = checkout scm
      version = "${scmVars.GIT_COMMIT}"
      fullName = "$branch-$version"

      pipeline.isEnabled()
    }

    stage('NPM'){
      container('node'){
          stage('build') {
            pipeline.npmInstall()
          }
      }
    }

    stage('Build container'){
      container('docker'){
        stage('Build'){
          pipeline.dockerBuildImage(imageName)
        }

        stage('Push to registry'){
          //pipeline.dockerPush('https://registry.hub.docker.com', 'docker-hub', env.BRANCH_NAME, version)
        }
      }
    }

    stage('Prepare deployment'){
      container('envsubst'){
        pipeline.prepareDeploy(branch, fullName)
      }
    }

    stage('Deploy to k8s cluster'){
      container('kubectl'){
        pipeline.deployToKubernetes(branch)
      }
    }

  }
}
