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

def deleteImageLocal(String urlRegistry, String repositoryRegistry, String tagApplication){
    sh 'docker rmi '+"${repositoryRegistry}"+':'+"${tagApplication}"
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
    def folderTerraform = 'terraform-laravel'
    def commitMsg = "chore(terraform-state): change terraform state "+"${BUILD_NUMBER}"

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
          sh 'git pull'    
        }           
      }
    }

    stage('Terraform Init') { 
      dir("${env.WORKSPACE}/${folderTerraform}"){
        sh 'docker run --user `id -u` -w /app -v `pwd`:/app hashicorp/terraform:light init'
      }
    }

    stage('Terraform Apply') { 
      dir("${env.WORKSPACE}/${folderTerraform}"){
        sh 'docker run --user `id -u` -w /app -v `pwd`:/app hashicorp/terraform:light apply -auto-approve'
      }
    }

    stage('Push TFState'){ 
    	dir("${env.WORKSPACE}/${folderTerraform}"){
	        withCredentials([usernamePassword(credentialsId: 'GITHUB', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
	        	sh 'git add -A'
	        	sh 'git config --global user.email "aguelsw@gmail.com"'
	        	sh 'git config --global user.name "Aguel Satria Wijaya"'
	        	try{
	        		sh 'git commit -m "'+"${commitMsg}"+'"'
	        	}catch(Exception e) {
	        		echo "Skip commit! Try push..."
	        	}
	        	try{
	            	sh 'git push https://'+"${GIT_USERNAME}"+":"+"${GIT_PASSWORD}"+'@github.com/'+"${GIT_USERNAME}"+'/'+"${folderTerraform}"+'.git'
	        	}catch(Exception e) {
	        		echo "Skip push! Next step..."
	        	}
	        }
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
        }
        deleteImageLocal(
            urlRegistry as String,
            repositoryRegistry as String,
            buildNumberApplication as String
        )
        deleteImageLocal(
            urlRegistry as String,
            repositoryRegistry as String,
            buildMustApplication as String
        )
    }
}