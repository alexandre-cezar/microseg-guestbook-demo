node {

    stage('Startup Process') {
        sh 'rm -f -r -d *'
        sh 'rm -f -r -d .[!.]* ..?*'
    }

    stage('cloneRepository') {
        checkout scm
    }

    stage('Deploy Guestbook App') {
        withKubeConfig([credentialsId: 'kubeconfig-cluster2',
                    caCertificate: '',
                    serverUrl: '',
                    contextName: '',
                    clusterName: '',
                    namespace: ''
                    ]) {
            sh "kubectl create namespace ${env.ns} --dry-run=client -o yaml | kubectl apply -f -  && \
            kubectl apply -f redis-leader-deployment.yaml -n ${env.ns} && \
            kubectl apply -f redis-leader-service.yaml -n ${env.ns} && \
            kubectl apply -f redis-follower-deployment.yaml -n ${env.ns} && \
            kubectl apply -f redis-follower-service.yaml -n ${env.ns} && \
            kubectl apply -f frontend-deployment.yaml -n ${env.ns} && \
            kubectl apply -f frontend-service.yaml -n ${env.ns}"
        }
    }

    stage('Get Microsegmentation Creds') {
        withCredentials([file(credentialsId: 'aporeto-creds', variable: 'mycreds')]) {
            sh "cp \$mycreds $WORKSPACE/default.creds"
        }       
    }

    stage('Create Ruleset for Guestbook App') {
        sh 'curl -o apoctl https://download.aporeto.com/apoctl/linux/apoctl'
        sh 'chmod +x apoctl'
        withEnv(["APOCTL_CREDS=$WORKSPACE/default.creds"]) {
            sh "./apoctl api import -n /${env.tenant}/${env.cloudAccount}/${env.group}/${env.ns} -f guestbook-ruleset.yaml \
                --set tenant=${env.tenant} --set cloudAccount=${env.cloudAccount} --set group=${env.group} --set ns=${env.ns}"
        }
    }

    stage('Finish Process') {
        sh 'rm -f -r -d *'
        sh 'rm -f -r -d .[!.]* ..?*'
    }
}