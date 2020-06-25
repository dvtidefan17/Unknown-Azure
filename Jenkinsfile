node {

    def app
	def app_name = 'staging/webapp'
	def app_image_tag = "${env.REPOSITORY}/${app_name}:v${env.BUILD_NUMBER}"

    stage('Clone GitHub repository') {
        /* Let's make sure we have the repository cloned to our workspace */
        checkout scm
    }

    stage('Build docker image') {
        /* This builds the actual image; synonymous to
         * docker build on the command line */

        app = docker.build($app_name)
    }

    stage('Test docker image') {
        /* Ideally, we would run a test framework against our image.
         * For this example, we're using a Volkswagen-type approach ;-) */
        app.inside {
            sh 'echo "Tests passed"'
        }
    }

    //Stage 4: Push the Image to a Docker Registry
	/* 
		Get Credentials using az acr credential show -n acrCICD17
		Set the DOCKER_CREDENTIALS_ID to the name used to stored the credentials
		The USERNAME and PASSWORD are pulled from that set of stored credentials
	*/
	stage('Push Docker Image to Docker Registry') {
		withCredentials([[$class: 'UsernamePasswordMultiBinding',
		credentialsId: env.DOCKER_CREDENTIALS_ID,
		usernameVariable: 'USERNAME',
		passwordVariable: 'PASSWORD']]) {
			docker.withRegistry(env.DOCEKR_REGISTRY, env.DOCKER_CREDENTIALS_ID) {
				app.push("${env.BUILD_NUMBER}")
				app.push("latest")
			}
		}
	}
	
	stage('Deploy Application on K8s') {
		/* 
			Get Credentials using az aks get-credentials --resource-group CICDResourceGroup --name CICDAKSCluser
			Set the K8s_CREDENTIALS_ID to the name used for the stored credential
			Set the K8s_CONTEXT_NAME CICDAKSCluser
			Set as an example - K8s_SERVER_URL https://cicdaksclu-cicdresourcegrou-561fa0-43151d27.hcp.westus2.azmk8s.io:443 use the value from the output of the 
			Set the K8s_CLUSTER_NAME to CICDAKSCluser
		*/
		withKubeConfig([credentialsId: env.K8s_CREDENTIALS_ID,
		serverUrl: env.K8s_SERVER_URL,
		contextName: env.K8s_CONTEXT_NAME,
		clusterName: env.K8s_CLUSTER_NAME]){
			sh 'cat deployment.yml | sed "s/{{BUILD_NUMBER}}/$BUILD_NUMBER/g" | kubectl apply -f -'
			sh 'kubectl apply -f service-node.yml'
		}     
    }
	
	stage('Cleanup Envirounment') {
        /* This removes the current docker build
         * and leaves latest in the local repository */
		sh "docker rmi ${env.BUILD_NUMBER}"
    }
}