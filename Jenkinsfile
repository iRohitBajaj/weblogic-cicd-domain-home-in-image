pipeline {
    options {
        disableConcurrentBuilds()
    }

    parameters {
        choice(name: 'DEPLOY_TYPE', choices: ['Create', 'Update'], description: 'Create new weblogic stack or apply updates to existing')
    }

    agent any

    environment {
        WLSIMG_BLDDIR = "${env.WORKSPACE}/resources/build"
        WLSIMG_CACHEDIR = "${env.WORKSPACE}/resources/cache"
        REQ_INSTALLERS_DIR = "${env.INSTALLERS_DIR}"
        IMAGE_NAME = "dockerish82/blog-domain-home-in-image:${sh(returnStdout: true, script: 'date +%Y%m%d%H%M')}"
    }

    stages {
        stage ('Environment') {
            steps {
                sh '''
                    mkdir -p  ${WLSIMG_BLDDIR} ${WLSIMG_CACHE_DIR}
                    echo "IMAGE_NAME = ${IMAGE_NAME}"
                    echo "PATH = ${PATH}"
                    echo "JAVA_HOME = ${JAVA_HOME}"
                '''
            }
        }
        stage ('Build New Image') {
            agent {
                     label 'master'
                }
            when {
                    expression {
                    DEPLOY_TYPE == 'Create'
                }
            }
            steps {
                sh '''
                    curl -SLO https://github.com/oracle/weblogic-image-tool/releases/download/release-1.9.4/imagetool.zip
                    unzip -o ./imagetool.zip
                    curl -SLO https://github.com/oracle/weblogic-deploy-tooling/releases/download/release-1.9.6/weblogic-deploy.zip
                    rm -rf ${WLSIMG_CACHEDIR}
                    imagetool/bin/imagetool.sh cache addInstaller --type wls --path "${REQ_INSTALLERS_DIR}"/fmw_12.2.1.4.0_wls_Disk1_1of1.zip --version 12.2.1.4.0
                    imagetool/bin/imagetool.sh cache addInstaller --type jdk --path "${REQ_INSTALLERS_DIR}"/jdk-8u212-linux-x64.tar.gz --version 8u212
                    imagetool/bin/imagetool.sh cache addInstaller --type wdt --version latest --path weblogic-deploy.zip
                    imagetool/bin/imagetool.sh create --tag ${IMAGE_NAME} --version 12.2.1.4.0 --jdkVersion=8u171 --wdtModel=BlogDomain-WDT-v1.yaml --wdtArchive=BlogDomain-WDT.zip --wdtDomainHome=/u01/oracle/user_projects/domains/sample-domain2 --wdtVariables=BlogDomain-WDT.properties --wdtDomainType WLS --chown oracle:root
                '''
            }
        }
        stage ('Update existing Image') {
            agent {
                     label 'master'
                }
            when {
                    expression {
                    DEPLOY_TYPE == 'Update'
                }
            }
            steps {
                sh '''
                    curl -SLO https://github.com/oracle/weblogic-image-tool/releases/download/release-1.9.4/imagetool.zip
                    unzip -o ./imagetool.zip
                    curl -SLO https://github.com/oracle/weblogic-deploy-tooling/releases/download/release-1.9.6/weblogic-deploy.zip
                    rm -rf ${WLSIMG_CACHEDIR}
                    imagetool/bin/imagetool.sh cache addInstaller --type wls --path "${REQ_INSTALLERS_DIR}"/fmw_12.2.1.4.0_wls_Disk1_1of1.zip --version 12.2.1.4.0
                    imagetool/bin/imagetool.sh cache addInstaller --type jdk --path "${REQ_INSTALLERS_DIR}"/jdk-8u212-linux-x64.tar.gz --version 8u212
                    imagetool/bin/imagetool.sh cache addInstaller --type wdt --version latest --path weblogic-deploy.zip
                    imagetool/bin/imagetool.sh update --tag ${IMAGE_NAME} --fromImage=dockerish82/blog-domain-home-in-image:v1 --wdtModel=BlogDomain-WDT-v1.yaml --wdtArchive=BlogDomain-WDT.zip --wdtDomainHome=/u01/oracle/user_projects/domains/sample-domain2 --wdtVariables=BlogDomain-WDT.properties --wdtDomainType WLS --chown oracle:root
                '''
            }
        }
        stage ('Push tagged Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-login', passwordVariable: 'DOCKER_REGISTRY_PWD', usernameVariable: 'DOCKER_REGISTRY_USER')]) {
                    sh '''
                        docker push ${IMAGE_NAME}
                    '''
                }
            }
        }
        stage('Deploy new application') {
            when {
                    expression {
                    DEPLOY_TYPE == 'Create'
                }
            }
            steps {
                sh '''
                    export PATH=/var/lib/jenkins/bin:$PATH
                    kubectl apply -f ./domain.yaml
                '''
            }
        }
        stage('Rolling updates') {
            when {
                    expression {
                    DEPLOY_TYPE == 'Update'
                }
            }
            steps {
                sh '''
                    export PATH=/var/lib/jenkins/bin:$PATH
                    kubectl patch domain sample-domain2 -n sample-domain2-ns --type='json' -p='[{"op" : "replace", "path": "/spec/image", "value": '"${IMAGE_NAME}"'}]' 
                '''
            }
        }
   }
   post {
    cleanup {
        deleteDir()
    }
  }
}
