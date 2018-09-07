#!groovy


// pod utilisé pour la compilation du projet
podTemplate(label: 'chart-run-pod', containers: [

        // le slave jenkins
        containerTemplate(name: 'jnlp', image: 'jenkinsci/jnlp-slave:alpine'),

        containerTemplate(name: 'helm', image: 'lachlanevenson/k8s-helm:v2.9.1', ttyEnabled: true, command: 'cat'),

        // un conteneur pour déployer les services kubernetes
        containerTemplate(name: 'kubectl', image: 'lachlanevenson/k8s-kubectl', command: 'cat', ttyEnabled: true)],

        // montage nécessaire pour que le conteneur docker fonction (Docker In Docker)
        volumes: [hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock')]
) {

    node('chart-run-pod') {

        properties([
                parameters([
                        string(defaultValue: 'latest', description: 'Version à déployer', name: 'image'),
                        string(defaultValue: '', description: 'Nom du chart à deployer', name: 'chart'),
                        string(defaultValue: '', description: 'URL de l application', name: 'alias'),
                        string(defaultValue: 'dev', description: 'Environnement de déploiement', name: 'env')
                ])
                ])

        stage('checkout sources') {
            checkout scm;
        }

        container('helm') {

            stage('upgrade') {
                withCredentials([string(credentialsId: 'registry_url', variable: 'registry_url')]) {

                    sh "helm init --client-only"

                    sh "helm repo add meltingpoc-charts https://softeamouest.github.io/charts"

                    def platform = params.env == 'prod' ? '' : '-' + params.env

                    def url = params.alias == '' : "${params.chart}${platform}.k8.wildwidewest.xyz" ? "${params.alias}${platform}.k8.wildwidewest.xyz"

                    def options = "--namespace ${params.env} --set-string env=${platform} --set-string image.tag=${params.image} meltingpoc-charts/${params.chart} --set ingress.hosts[0]=${url},ingress.tls[0].hosts[0]=${url}"

                    sh "if [ `helm list --namespace ${params.env} | grep ^${params.chart} | wc -l` == '0' ]; then helm install --name ${params.chart} ${options}; fi"

                    sh "if [ `helm list --namespace ${params.env} | grep ^${params.chart} | wc -l` == '1' ]; then helm upgrade ${params.chart} ${options}; fi"
                }
            }
        }
    }
}
