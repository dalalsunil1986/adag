def label = "mypod-${UUID.randomUUID().toString()}"
def serviceaccount = "cicd-jenkins"
podTemplate(label: label, serviceAccount: serviceaccount,
containers: [containerTemplate(name: 'kubectl', image: 'smesch/kubectl', ttyEnabled: true, command: 'cat',
                             volumes: [secretVolume(secretName: 'kube-config', mountPath: '/root/.kube')]),
    containerTemplate(name: 'docker', image: 'docker:19.03', ttyEnabled: true, command: 'cat')],
                             volumes: [hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock')]
) {

    node(label) {
        def GIT_URL= 'https://github.com/somatiwari2005/adv_devops.git'
		def GIT_CREDENTIAL_ID ='scm'
		def GIT_BRANCH='master'
		def DOCKER_HUB_ACCOUNT = '11480778'
        def DOCKER_IMAGE_NAME = 'sample-image'
        def K8S_DEPLOYMENT_NAME = 'sample-app'
		
        stage('Clone Repo'){
			git branch: GIT_BRANCH, url: GIT_URL, credentialsId: GIT_CREDENTIAL_ID
        }

        stage('Docker Build') {
			container('docker'){
			   sh("docker build -t ${DOCKER_HUB_ACCOUNT}/${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER} --network=host .")
			}
		}

        stage('Push Container Image') {
			container('docker'){
				withDockerRegistry([ credentialsId: "dockerhub", url: "" ]) {
				   sh("docker push ${DOCKER_HUB_ACCOUNT}/${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}")
				}
			}
		}
		
        stage('Deploy build to Kubernetes') {
            container('kubectl') {
                try{
                    sh("kubectl get deployment/django-st -n default")
                    if(true){
                        sh ("kubectl set image deployment/${K8S_DEPLOYMENT_NAME} ${K8S_DEPLOYMENT_NAME}=${DOCKER_HUB_ACCOUNT}/${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER} -n default")
                    }
                }
                catch(e){
                    sh("kubectl run ${K8S_DEPLOYMENT_NAME} --image=${DOCKER_HUB_ACCOUNT}/${DOCKER_IMAGE_NAME}:latest --replicas=2 -n default")
                    echo "deploying"
                }
                sh ("kubectl get pods -n default")
            }
        }
	}
 }
