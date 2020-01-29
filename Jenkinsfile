def label = "mypod-${UUID.randomUUID().toString()}"
def serviceaccount = "jenkins-admin"
podTemplate(label: label, serviceAccount: serviceaccount,
containers: [containerTemplate(name: 'kubectl', image: 'smesch/kubectl', ttyEnabled: true, command: 'cat',
                             volumes: [secretVolume(secretName: 'kube-config', mountPath: '/root/.kube')]),
    containerTemplate(name: 'docker', image: 'docker:19.03', ttyEnabled: true, command: 'cat')],
                             volumes: [hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock')]
) {

    node(label) {
        def GIT_URL= 'https://innersource.accenture.com/scm/adv-devops/advanced_devops.git'
		def GIT_CREDENTIAL_ID ='scm-credentials-st'
		def GIT_BRANCH='master'
        
        stage ('Git Checkout'){
			git branch: GIT_BRANCH, url: GIT_URL, credentialsId: GIT_CREDENTIAL_ID
        }
		
        stage ('Docker Build') {
			/*container('docker'){
				app = docker.build('nexgtech/django')
			}*/
          container('docker'){
			    sh("docker build -t nexgtech/django --network=host .")
			}
		}

        stage ('Push Image') {
			container('docker'){
				docker.withRegistry('https://registry.hub.docker.com', 'docker-container') {
					//docker.image('django').push("${env.BUILD_ID}")
					app.push("${env.BUILD_NUMBER}")
					app.push("latest")
				}
			}
		}
        stage('Deploy build to Kubernetes') {
            container('kubectl') {
                try{
                    sh("kubectl get deployment/django -n default")
                    if(true){
                        sh ("kubectl set image deployment/django django=nexgtech/django:${env.BUILD_ID} -n default") 
                    }
                } 
                catch(e){
                    sh("kubectl run django --image=nexgtech/django:latest --replicas=2 -n default")
                    echo "deploying"
                }
                sh ("kubectl get pods -n default")
            }
        } 
	}
 }
