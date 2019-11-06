// Declarative Jenkinsfile Pipeline for a Hashicorp packer/terraform AWS simple ec2 stack
// (n.b. use of env.BRANCH_NAME to filter stages based on branch means this needs to be part
// of a Multibranch Project in Jenkins - this fits with the model of branches/PR's being
// tested & master being deployed)
pipeline {
  agent any
  environment {
     AWS_DEFAULT_REGION = 'us-east-1'
  }

  stages {
    stage('Validate & Lint Packer Syntax') {
      parallel {
        stage('packer validate') {
          agent {
            docker {
              image 'simonmcc/hashicorp-pipeline:latest'
              alwaysPull false
            }
          }
          steps {
            checkout scm
            wrap([$class: 'AnsiColorBuildWrapper', 'colorMapName': 'xterm']) {
              sh "packer validate ./base/rhel-7.json"
            }
          }
        }
        stage('Verify Terraform') {
          agent { docker { image 'simonmcc/hashicorp-pipeline:latest' } }
          steps {
            checkout scm
            sh "terraform fmt -check=true -diff=true"
          }
        }
      }
    }
    stage('Build Images - Apply Chef Cookbook') {
      agent { docker { image 'simonmcc/hashicorp-pipeline:latest' } }
      steps {
        checkout scm
        withCredentials([[$class: 'AmazonWebServicesCredentialsBinding',
                          credentialsId: 'demo-aws-creds',
                          accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                          secretKeyVariable: 'AWS_SECRET_ACCESS_KEY' ]]) {
          wrap([$class: 'AnsiColorBuildWrapper', 'colorMapName': 'xterm']) {
            sh "echo 'Building Image'"
            sh "echo 'Applying Chef Cookbook'"
          }
        }
      }
    }

    stage('Test Image') {
      agent { docker { image 'simonmcc/hashicorp-pipeline:latest' } }
      when {
        expression { env.BRANCH_NAME != 'master' }
      }
      steps {
        checkout scm
        withCredentials([[$class: 'AmazonWebServicesCredentialsBinding',
                          credentialsId: 'demo-aws-creds',
                          accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                          secretKeyVariable: 'AWS_SECRET_ACCESS_KEY' ]]) {
          wrap([$class: 'AnsiColorBuildWrapper', 'colorMapName': 'xterm']) {
            sh "echo 'run Terraform plan'"
            sh "echo 'apply Terraform plan'"
            sh "echo 'cat output.json'"
          }
        }
      }
    }
    stage('Verify Image Compliance - InSpec') {
      agent {
        docker {
          image 'chef/inspec:latest'
           args "--entrypoint=''"
        }
      }
      when {
        expression { env.BRANCH_NAME != 'master' }
      }
      steps {
        checkout scm
        withCredentials([[$class: 'AmazonWebServicesCredentialsBinding',
                          credentialsId: 'demo-aws-creds',
                          accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                          secretKeyVariable: 'AWS_SECRET_ACCESS_KEY' ]]) {
          wrap([$class: 'AnsiColorBuildWrapper', 'colorMapName': 'xterm']) {
            sh "echo 'Running InSpec Tests'"
            
          }
        }
      }
    }
    stage('Destroy Test Stack') {
      agent { docker { image 'simonmcc/hashicorp-pipeline:latest' } }
      when {
        expression { env.BRANCH_NAME != 'master' }
      }
      steps {
        checkout scm
        withCredentials([[$class: 'AmazonWebServicesCredentialsBinding',
                          credentialsId: 'demo-aws-creds',
                          accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                          secretKeyVariable: 'AWS_SECRET_ACCESS_KEY' ]]) {
          wrap([$class: 'AnsiColorBuildWrapper', 'colorMapName': 'xterm']) {
            sh "echo 'Destroying test stack'"
          }
        }
      }
    }
    stage('Deploy Image to Terraform') {
      agent { docker { image 'simonmcc/hashicorp-pipeline:latest' } }
      when {
        expression { env.BRANCH_NAME == 'master' }
      }
      steps {
        checkout scm
        withCredentials([[$class: 'AmazonWebServicesCredentialsBinding',
                          credentialsId: 'demo-aws-creds',
                          accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                          secretKeyVariable: 'AWS_SECRET_ACCESS_KEY' ]]) {
          wrap([$class: 'AnsiColorBuildWrapper', 'colorMapName': 'xterm']) {
            sh "echo 'Deploying Image to Terraform'"
          }
        }
      }
    }
    stage('Manual Approval for Terraform Plan') {
      when {
        expression { env.BRANCH_NAME == 'master' }
      }
      steps {
        input 'Do you approve the apply?'
      }
    }
    stage('Terraform Apply - Master Branch') {
      agent { docker { image 'simonmcc/hashicorp-pipeline:latest' } }
      when {
        expression { env.BRANCH_NAME == 'master' }
      }
      steps {
        checkout scm
        withCredentials([[$class: 'AmazonWebServicesCredentialsBinding',
                          credentialsId: 'demo-aws-creds',
                          accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                          secretKeyVariable: 'AWS_SECRET_ACCESS_KEY' ]]) {
          wrap([$class: 'AnsiColorBuildWrapper', 'colorMapName': 'xterm']) {
            sh "echo 'Finished applying production Terraform plan'"
          }
        }
      }
    }
  }
}
