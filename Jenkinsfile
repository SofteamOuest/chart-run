#!groovy
import java.text.SimpleDateFormat

// pod utilisé pour la compilation du projet
podTemplate(label: 'chart-run-pod', nodeSelector: 'medium', containers: [

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
                ]),
                buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '',
                        daysToKeepStr: '1', numToKeepStr: '3')),
                pipelineTriggers([])])

        stage('checkout sources') {
            checkout scm;
        }

        container('helm') {

            stage('upgrade') {
                withCredentials([string(credentialsId: 'registry_url', variable: 'registry_url')]) {

                    sh "helm init --client-only"

                    sh "helm repo add meltingpoc-charts https://softeamouest.github.io/charts"

                    sh "if [ `helm list | grep ^${params.chart} | wc -l` == '0' ]; then helm install --name ${params.env != '' ? params.chart + params.env : params.chart} --set-string image.tag=${params.image} meltingpoc-charts/${params.chart}; fi"

                    sh "if [ `helm list | grep ^${params.chart} | wc -l` == '1' ]; then helm upgrade ${params.env != '' ? params.chart + params.env : params.chart} --set-string image.tag=${params.image} meltingpoc-charts/${params.chart}; fi"
                }
            }
        }
    }
}