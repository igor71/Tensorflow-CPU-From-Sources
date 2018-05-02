pipeline {
   agent {label 'Tensorflow-Project'}
      stages {
         stage('Clone Tensorflow Repository') {
            steps {
             sh '''#!/bin/bash -xe
             export TF_BRANCH=r1.7
             cd /
             echo 'jenkins' | sudo -S rm -rf tensorflow
             echo 'jenkins' | sudo -S git clone --branch=${TF_BRANCH} --depth=1 https://github.com/tensorflow/tensorflow.git
             cd tensorflow
             echo 'jenkins' |sudo -S git checkout ${TF_BRANCH}
             echo 'jenkins' |sudo -S updatedb
                '''
            }
    }
         stage('Configure Build ENV & Build TensorFlow Package From Sources') {
            steps {
             sh '''#!/bin/bash -xe
                   cd /          
                   pwd
                   echo 'jenkins' | sudo -S cp build_tf_package.sh /tensorflow
                   cd tensorflow
                   echo 'jenkins' | sudo -S bash build_tf_package.sh
                '''
            }
    }
         stage('Install Tensorflow Package') {
            steps {
                  sh '''#!/bin/bash -xe
                  echo 'jenkins' | sudo -S pip --no-cache-dir install --upgrade ${WHL_DIR}/tensorflow-*.whl
                  echo 'jenkins' | sudo -S rm -rf /root/.cache
                     '''
            }
     }
         stage('Testing Tensorflow Installation') {
            steps {
             sh '''#!/bin/bash -xe
                   cd /
                   python cpu_tf_check.py && python unitest.py 
                       if [ "$?" != "0" ]; then
                          echo "Tensorflow build Failed!!!"
                          exit -1
                       fi
                 ''' 
            }
    }
         stage('Push Arifact To Network Share') {
            steps {
             sh '''#!/bin/bash -xe
                   export TFLOW=$(cd ${WHL_DIR} && find -type f -name "tensorflow*.whl" | cut -c 3-)
                   pv ${WHL_DIR}/${TFLOW} > /media/common/IT/${TFLOW}
                   cd ${WHL_DIR}
                   md5sum /media/common/IT/${TFLOW} > ${TFLOW}.md5
                   md5sum -c ${TFLOW}.md5 
                       if [ "$?" != "0" ]; then
                          echo "SHA1 changed! Security breach? Job Will Be Marked As Failed!!!"
                          exit -1
                       fi
                 ''' 
            }
    }
         stage('Remove Build Folder') {
            steps {
             sh 'sudo -S rm -rf ${WHL_DIR}'
            }
     }
}      
         post {
            always {
               script {
                  if (currentBuild.result == null) {
                     currentBuild.result = 'SUCCESS' 
                  }
               }
               step([$class: 'Mailer',
                     notifyEveryUnstableBuild: true,
                     recipients: "igor.rabkin@xiaoyi.com",
                     sendToIndividuals: true])
            }
         } 
}
