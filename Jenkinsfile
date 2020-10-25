pipeline {
    options {
        disableConcurrentBuilds()
    }
    agent any
    tools {
        jdk 'jdk8'
    }

    environment {
        WLSIMG_BLDDIR = "${env.WORKSPACE}/resources/build"
        WLSIMG_CACHEDIR = "${env.WORKSPACE}/resources/cache"
        IMAGE_NAME = "dockerish82/blog-domain-home-in-image:v1"
    }

    stages {
        stage ('Environment') {
            steps {
                sh '''
                    mkdir -p  ${WLSIMG_BLDDIR} ${WLSIMG_CACHE_DIR}
                    echo "IMAGE_NAME = ${IMAGE_NAME}"
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
                    echo "JAVA_HOME = ${JAVA_HOME}"
                '''
            }
        }
        stage ('Build Image') {
            steps {
                sh '''
                    curl -SLO https://github.com/oracle/weblogic-image-tool/releases/download/release-1.9.4/imagetool.zip
                    unzip -o ./imagetool.zip
                    curl -SLO https://github.com/oracle/weblogic-deploy-tooling/releases/download/release-1.9.6/weblogic-deploy.zip
                    rm -rf ${WLSIMG_CACHEDIR}
                    imagetool/bin/imagetool.sh cache addInstaller --type wdt --version latest --path weblogic-deploy.zip
                    imagetool/bin/imagetool.sh create --tag ${IMAGE_NAME} --fromImage dockerish82/weblogicrepo:12.2.1.4 --wdtModel=./BlogDomain-WDT-v1.yaml --wdtArchive=./BlogDomain-WDT.zip --wdtDomainHome=/u01/oracle/user_projects/domains/sample-domain2 --wdtVariables=./BlogDomain-WDT.properties --wdtDomainType WLS --chown oracle:root 
                '''
            }
        }
        stage ('Push tagged Image') {
            steps {
                sh '''
                    docker push ${IMAGE_NAME}
                '''
            }
        }
        stage('Roll Updates') {
            steps {
                sh '''
                    export PATH=/var/lib/jenkins/bin:$PATH
                    kubectl apply -f ./domain.yaml
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
