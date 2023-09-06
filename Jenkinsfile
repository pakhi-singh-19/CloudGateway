node {
    def repourl = "${REGISTRY_URL}/${PROJECT_ID}/${ARTIFACT_REGISTRY}/cloud-gateway:${env.BUILD_NUMBER}"
    def mvnHome = tool name: 'maven', type: 'maven'
    def mvnCMD = "${mvnHome}/bin/mvn "

    stage('checkout') {
        checkout([$class: 'GitSCM',
            branches: [[name: '*/master']],
            extensions: [],
            userRemoteConfigs: [[credentialsId: 'pakhi-singh-19', url: 'https://github.com/pakhi-singh-19/CloudGateway.git']]])
    }

    stage('Build and Push Docker Image') {
        withCredentials([file(credentialsId: 'gcp', variable: 'GC_KEY')]) {
            sh("gcloud auth activate-service-account --key-file=${GC_KEY}")
            sh 'gcloud auth configure-docker us-central1-docker.pkg.dev'
            sh "${mvnCMD} clean install jib:build -DREPO_URL=${REGISTRY_URL}/${PROJECT_ID}/${ARTIFACT_REGISTRY}/cloud-gateway:${env.BUILD_NUMBER}"
        }
    }

    stage('Deploy') {
        script {
            sh "sed -i 's|IMAGE_URL|${repourl}|g' deployment.yaml"
        }
        step([$class: 'KubernetesEngineBuilder',
            projectId: env.PROJECT_ID,
            clusterName: env.CLUSTER_NAME,
            location: env.ZONE,
            manifestPattern: 'deployment.yaml',
            credentialsId: env.PROJECT_ID,
            verifyDeployments: true])
    }
}
