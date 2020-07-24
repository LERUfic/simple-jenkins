def checkout(String repositoryGit, String credentialsGit, String branchGit){
	checkout([
		$class: 'GitSCM',
		branches: [[name: branchGit]],
		doGenerateSubmoduleConfigurations:false,
		extensions: [],
		submoduleCfg: [],
		userRemoteConfigs: [[credentialsId: credentialsGit,url: repositoryGit]]
	])
}

def buildImagePush(String urlRegistry, String credentialRegistry, String repositoryRegistry, String tagApplication){
    def appImg
	script {
		docker.withRegistry(urlRegistry,credentialRegistry){
			appImg = docker.build(repositoryRegistry+':'+"${tagApplication}",'.')
			appImg.push()
		}
	}
}

node {
	def nameApplication = 'laravel'
	def buildNumberApplication = 'v'+"${BUILD_NUMBER}"
	def buildMustApplication = 'latest'
	
	def repositoryRegistry = 'lerufic/laravel'
	def credentialRegistry = 'b9aa42f2-5996-49f0-b863-e6c729a1a142'
	def urlRegistry = 'https://registry.hub.docker.com'

	def repositoryGit = 'https://github.com/LERUfic/laravel-docker'
	def branchGit = '*/master'
	def credentialsGit = '805eea8d-f153-4916-9800-b5c4af7b45bb'

	stage('Checkout Git'){
	    checkout(
	        repositoryGit as String,
	        credentialsGit as String,
	        branchGit as String
		)
	}
	
	stage('Build by Tag'){
	    buildImagePush(
	        urlRegistry as String,
	        credentialRegistry as String,
	        repositoryRegistry as String,
	        buildNumberApplication as String
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
	
	stage('Cleanup Docker'){
		dir("."){
			sh 'docker image prune --all'
		}
	}
}