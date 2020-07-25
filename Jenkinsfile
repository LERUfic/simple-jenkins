def checkout(String repositoryGit, String credentialsGit, String branchGit){
    checkout([
        $class: 'GitSCM',
        branches: [[name: branchGit]],
        doGenerateSubmoduleConfigurations:false,
        extensions: [[$class: 'RelativeTargetDirectory',
                       relativeTargetDir: 'MyGit/']],
        submoduleCfg: [],
        userRemoteConfigs: [[credentialsId: credentialsGit,url: repositoryGit]]
    ])
}

def buildImagePush(String urlRegistry, String credentialRegistry, String repositoryRegistry, String tagApplication){
    def appImg
    script {
        docker.withRegistry(urlRegistry,credentialRegistry){
            appImg = docker.build(repositoryRegistry+':'+"${tagApplication}",'MyGit/')
            appImg.push()
        }
    }
}

node {
    def nameApplication = 'laravel'
    def buildNumberApplication = 'v'+"${BUILD_NUMBER}"
    def buildMustApplication = 'latest'
    
    def repositoryRegistry = 'lerufic/laravel'
    def credentialRegistry = 'REGISTRY'
    def urlRegistry = 'https://registry.hub.docker.com'

    def repositoryGit = 'https://github.com/LERUfic/laravel-docker'
    def branchGit = '*/master'
    def credentialsGit = 'GITHUB'

    def repositoryTerraform = 'https://github.com/LERUfic/terraform-laravel'

    stage('Checkout Git'){
        checkout(
            repositoryGit as String,
            credentialsGit as String,
            branchGit as String
        )
    }

    stage('Build for Latest Tag'){
        buildImagePush(
            urlRegistry as String,
            credentialRegistry as String,
            repositoryRegistry as String,
            buildMustApplication as String
        )
    }

    stage('Clone Infrastructure') {
      dir("${env.WORKSPACE}"){
        sh 'docker pull hashicorp/terraform:light'
      }
      try {
        dir("${env.WORKSPACE}"){
          sh 'git clone '+"${repositoryTerraform}"    
        }
      }catch(Exception e) {
        dir("${env.WORKSPACE}/terraform-laravel"){
          sh 'cd git'
          sh 'git pull'    
        }           
      }
    }

    stage('Terraform Init') { 
      dir("${env.WORKSPACE}/terraform-laravel"){
        sh 'docker run --user `id -u` -w /app -v `pwd`:/app hashicorp/terraform:light init'
      }
    }

    stage('Terraform Apply') { 
      dir("${env.WORKSPACE}/terraform-laravel"){
        sh 'docker run --user `id -u` -w /app -v `pwd`:/app hashicorp/terraform:light apply -auto-approve'
      }
    }

    stage('Build by Tag'){
        buildImagePush(
            urlRegistry as String,
            credentialRegistry as String,
            repositoryRegistry as String,
            buildNumberApplication as String
        )
    }
    
    stage('Cleanup Docker'){
        dir("${env.WORKSPACE}"){
        	sh 'rm -rf MyGit/'
            sh 'docker image prune --all'
        }
    }
}