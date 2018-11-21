def label = "codeone-${UUID.randomUUID().toString()}"
podTemplate(
    name: 'codeone',
    label: label,
    inheritFrom: 'default',
    containers: [
        containerTemplate(
            name: 'golang', 
            image: 'golang:1.11-alpine',
            ttyEnabled: true,
            command: 'cat'
        ),
        containerTemplate(
            name: 'kaniko', 
            image: 'dcurrie/kaniko:alpine',
            ttyEnabled: true,
            command: 'cat'
        ),
        containerTemplate(
            name: 'helm', 
            image: 'lachlanevenson/k8s-helm:v2.11.0',
            ttyEnabled: true,
            command: 'cat'
        )
    ]
) {
    node(label) {
        def commitId
        stage ('Extract') {
            commitId = checkout(scm).GIT_COMMIT.take(8)
        }
        stage ('Build') {
            container ('golang') {
                sh 'CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main .'
            }
        }
        def repository
        stage ('Docker') {
            container ('kaniko') {
				withCredentials([[$class: 'UsernamePasswordMultiBinding',
					credentialsId: '123456',
					usernameVariable: 'DOCKER_HUB_USER',
					passwordVariable: 'DOCKER_HUB_PASSWORD']]){
				sh "executor -f `pwd`/Dockerfile -c `pwd` --insecure --skip-tls-verify --cache=true --destination=${env.DOCKER_HUB_USER}/kaniko:${env.BUILD_NUMBER}"
				}
            }
        }
        stage ('Deploy') {
            container ('helm') {
                sh """helm init --client-only --skip-refresh
                helm upgrade --install --wait --set image.repository=${repository},image.tag=${commitId} hello hello"""
            }
        }
    }
}