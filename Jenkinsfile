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
                   echo 'jenkins' | sudo -S cp test.sh /tensorflow
                   cd tensorflow
                   echo 'jenkins' | sudo -S bash test.sh 
                '''
            }
    }
         stage('Install Tensorflow Package') {
            steps {
                  sh '''#!/bin/bash -xe
                  mv /home/jenkins/tensorflow-*.whl $WORKSPACE
                  echo 'jenkins' | sudo -S pip --no-cache-dir install --upgrade $WORKSPACE/tensorflow-*.whl
                  echo 'jenkins' | sudo -S rm -rf /root/.cache
                     '''
            }
     }
         stage('Testing Tensorflow Installation') {
            steps {
             sh '''#!/bin/bash -xe
                   cd /
                   echo 'jenkins' | sudo -S python cpu_tf_check.py && python unitest.py 
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
                   export TFLOW=$(cd $WORKSPACE && find -type f -name "tensorflow*.whl" | cut -c 3-)
                   pv $WORKSPACE/${TFLOW} > /media/common/IT/${TFLOW}
                   cd $WORKSPACE
                   md5sum /media/common/IT/${TFLOW} > ${TFLOW}.md5
                   md5sum -c ${TFLOW}.md5 
                       if [ "$?" != "0" ]; then
                          echo "SHA1 changed! Security breach? Job Will Be Marked As Failed!!!"
                          exit -1
                       fi
                 ''' 
            }
    }
         stage('Clean Build Folder') {
            steps {
             sh '''#!/bin/bash -xe
                   cd $WORKSPACE
                   rm -rf tensorflow-*
                 ''' 
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
