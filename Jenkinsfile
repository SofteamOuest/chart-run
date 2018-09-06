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
                        string(defaultValue: 'latest', description: 'version à déployer', name: 'image'),
                        string(defaultValue: '', description: 'chart à déployer', name: 'chart'),
                        string(defaultValue: '', description: 'env', name: 'env')
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

                    def chartName = params.env == '' ? params.chart : params.chart + "-" + params.env

                    def namespace = params.env == '' ? 'default' : params.env

                    def platform = params.env == 'prod' ? '' : '-' + params.env

                    def options = "--namespace ${namespace} --set-string env=platform --set-string image.tag=${params.image} meltingpoc-charts/${params.chart}"

                    sh "if [ `helm list --namespace ${namespace} | grep ^${chartName} | wc -l` == '0' ]; then helm install --name ${chartName} ${options}; fi"

                    sh "if [ `helm list --namespace ${namespace} | grep ^${chartName} | wc -l` == '1' ]; then helm upgrade ${chartName} ${options}; fi"
                }
            }
        }
    }
}
