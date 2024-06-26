
def DOCKER_REGISTRY_USER = 'src32'
def AMD64_LOCAL_IMAGE = 'ibm-bank-portal-amd64'
def TAG = 'v0.1'
def ENVIRONMENT = 'production'

node {
  try {
    stage('Checkout') {
      sh "echo ${DOCKER_REGISTRY_USER}"
      sh "echo ${AMD64_LOCAL_IMAGE}"
      sh "echo ${TAG}"
      checkout scm
    }

    stage('compile') {
      withMaven {
        sh 'mvn clean install'
      }

      dir('target') {
        sh 'cp bank-portal-1.0-SNAPSHOT.war ../build'
      }
    }

    stage('Build amd64 image') {
      dir('build') {
        sh "docker build -t ${DOCKER_REGISTRY_USER}/${AMD64_LOCAL_IMAGE}:${TAG} --platform=linux/amd64 ."
      }
    }

    stage('Push image') {
      docker.withRegistry('', 'docker-src32-accesstoken') {
        sh "docker push ${DOCKER_REGISTRY_USER}/${AMD64_LOCAL_IMAGE}:${TAG}"
      }
    }

        /** Start ---- Pre-integration Tests --------------- */
    /*stage('Pre-integration Tests') {
      parallel(

                'Quality Tests': {
                    //  sh "docker run --rm portal:v0.3-amd64"
                    sh 'podman run --publish-all --rm --name portal -d portal:v0.3-s390x'

                       sh 'chmod +x ./build/scripts/test.sh'
                       sh './build/scripts/test.sh'
                },
                'Unit Tests': {
                  sh 'chmod +x ./build/scripts/test.sh'
                  sh './build/scripts/test.sh'
                }
            )
    }*/

    stage('Deploy Image') {
      if (env.BRANCH_NAME == 'development' || env.BRANCH_NAME == 'preproduction') {
        dir('deploy') {
          echo 'Deploying to development or pre-production on S390X environment'
          sh "oc login --insecure-skip-tls-verify --token=${OCP_S390X_TOKEN} --server=${OCP_S390X_API_SERVER}"
          createOCPNamespaceIfNotExist('staging-deployment')

          deleteOCPDeploymentSVCRouteIfExist()
          sh 'oc create -f bank-portal.yaml' // & deployment config
          sh 'oc expose service portal' // Push new stream

                 /** IBM CLOUD - X86 ENVIRONMENT */
          echo 'Deploying to development or pre-production on IBM Cloud on X86 environment'
          sh "oc login --insecure-skip-tls-verify --token=${OCP_AMD64_TOKEN} --server=${OCP_AMD64_API_SERVER}"
          createOCPNamespaceIfNotExist('staging-deployment')

          deleteOCPDeploymentSVCRouteIfExist()
          sh 'oc create -f bank-portal.yaml' // & deployment config
          sh 'oc expose service portal' // Push new stream
        }
      }
        if (ENVIRONMENT == 'production') {
        timeout(time: 2, unit: 'HOURS') {
            input message: 'Approve Deploy?', ok: 'Yes'
            echo 'Your request to deploy this release to poduction is approved'
            dir('deploy') {
                /** LINUXONE/ONPREM - S390X ENVIRONMENT */
                sh "oc login --insecure-skip-tls-verify --token=${OCP_S390X_TOKEN} --server=${OCP_S390X_API_SERVER}"
            createOCPNamespaceIfNotExist('production-deployment')

                deleteOCPDeploymentSVCRouteIfExist()
                sh 'oc create -f bank-portal-prod.yaml' // & deployment config
                sh 'oc expose service portal' // Push new stream

               /** IBM CLOUD - X86 ENVIRONMENT */

            sh "oc login --insecure-skip-tls-verify --token=${OCP_AMD64_TOKEN} --server=${OCP_AMD64_API_SERVER}"
            createOCPNamespaceIfNotExist('production-deployment')

                deleteOCPDeploymentSVCRouteIfExist()
                sh 'oc create -f bank-portal-prod.yaml' // & deployment config
                sh 'oc expose service portal' // Push new stream
            }
        }
        }
    }
     } finally {
    stage('cleanup') {
      echo 'Clean up workspace'
        deleteDir()
    // sh 'docker volume rm $(docker volume ls -qf dangling=true)'
    }
  }
}
