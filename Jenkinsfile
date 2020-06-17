node {

    def app

    stage('Clone GitHub repository') {
        /* Let's make sure we have the repository cloned to our workspace */
        checkout scm
    }

    stage('Build docker image') {
        /* This builds the actual image; synonymous to
         * docker build on the command line */

        app = docker.build("staging/webapp")
    }

    stage('Test docker image') {
        /* Ideally, we would run a test framework against our image.
         * For this example, we're using a Volkswagen-type approach ;-) */

        app.inside {
            sh 'echo "Tests passed"'
        }
    }

    //Stage 6: Push the Image to a Docker Registry
	stage('Push Docker Image to Docker Registry') {
		container('docker'){
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
	}
	
	//Stage 7: Deploy Application on K8s
	stage('Deploy Application on K8s') {
		container('kubectl'){
			withKubeConfig([credentialsId: env.K8s_CREDENTIALS_ID,
			serverUrl: env.K8s_SERVER_URL,
			contextName: env.K8s_CONTEXT_NAME,
			clusterName: env.K8s_CLUSTER_NAME]){
				sh 'cat deployment.yml | sed "s/{{BUILD_NUMBER}}/$BUILD_NUMBER/g" | kubectl apply -f -'
				sh 'kubectl apply -f service-node.yml'
			}     
		}
    }
	
	stage('Cleanup Envirounment') {
        /* This removes the current docker build
         * and leaves latest in the local repository */
		sh "docker rmi localhost:5000/staging/webapp:$BUILD_NUMBER"
    }
}