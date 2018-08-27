#!groovy​
podTemplate(label: 'nginx-app', containers: [
    containerTemplate(name: 'kubectl', image: 'smesch/kubectl', ttyEnabled: true, command: 'cat',
        volumes: [secretVolume(secretName: 'kube-config', mountPath: '/root/.kube')]),
    containerTemplate(name: 'docker', image: 'docker', ttyEnabled: true, command: 'cat',
        envVars: [containerEnvVar(key: 'DOCKER_CONFIG', value: '/tmp/'),])],
        volumes: [secretVolume(secretName: 'docker-config', mountPath: '/tmp'),
                  hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock')
  ]) {

    node('nginx-app') {

        def DOCKER_HUB_ACCOUNT = 'kaarthikdocker'
        def DOCKER_IMAGE_NAME = 'nginx-app'
        def K8S_DEPLOYMENT_NAME = 'nginx-app'

        stage('Clone nginx App Repository') {
            checkout scm
 
            container('docker') {
                stage('Docker Build & Push Current & Latest Versions') {
                    sh ("docker build -t ${DOCKER_HUB_ACCOUNT}/${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER} .")
                    sh ("docker push ${DOCKER_HUB_ACCOUNT}/${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}")
                    sh ("docker tag ${DOCKER_HUB_ACCOUNT}/${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER} ${DOCKER_HUB_ACCOUNT}/${DOCKER_IMAGE_NAME}:latest")
                    sh ("docker push ${DOCKER_HUB_ACCOUNT}/${DOCKER_IMAGE_NAME}:latest")
                }
            }

               }
            container('kubectl') {
                stage('Deploy New Build To Kubernetes') {
                    sh ("kubectl run ${K8S_DEPLOYMENT_NAME} --image=${DOCKER_HUB_ACCOUNT}/${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER} --port=80")
		    sh ("kubectl scale deployment/${K8S_DEPLOYMENT_NAME} --replicas=3")
		    sh ("kubectl label deployment/${K8S_DEPLOYMENT_NAME} app=nginx-app")
		    sh ("kubectl expose deployment/${K8S_DEPLOYMENT_NAME} --port=80 --target-port=80 --name=nginx --type=LoadBalancer")
             }

        }        
    }
}
