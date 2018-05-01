pipeline {
   agent {label 'yi-tensorflow'}
      stages {
         stage('Clone Tensorflow Repository') {
            steps {
             sh '''#!/bin/bash -xe
             TF_BRANCH=r1.7
             cd /
             sudo -S git clone --branch=${TF_BRANCH} --depth=1 https://github.com/tensorflow/tensorflow.git
             cd tensorflow
             sudo -S git checkout ${TF_BRANCH}
             sudo -S updatedb
                '''
            }
    }
         stage('Configure Build & Bazel ENV') {
            steps {
             sh '''#!/bin/bash -xe
             export WHL_DIR=/whl
             export CI_BUILD_PYTHON=python PYTHON_BIN_PATH=/usr/bin/python PYTHON_LIB_PATH=/usr/local/lib/python2.7/dist-packages
             export CC_OPT_FLAGS='-march=native' TF_NEED_JEMALLOC=0 TF_NEED_GCP=0 TF_NEED_CUDA=0 TF_NEED_CUDA=0 TF_NEED_HDFS=0
             export TF_NEED_S3=0 TF_NEED_OPENCL=0 TF_NEED_GDR=0 TF_ENABLE_XLA=0 TF_NEED_VERBS=0 TF_NEED_MPI=0
             export TF_NEED_KAFKA=0 TF_NEED_OPENCL_SYCL=0
             yes N |./configure
                '''
            }
    }
         stage('Build TensorFlow Package From Sources') {
            steps {
             sh '''#!/bin/bash -xe
             export WHL_DIR=/whl
             sudo -S bazel build --config="opt" \
                              --config=mkl \
                              --copt="-DEIGEN_USE_VML" \
                              --cxxopt="-D_GLIBCXX_USE_CXX11_ABI=0" \
                              //tensorflow/tools/pip_package:build_pip_package
             sudo -S mkdir ${WHL_DIR}
             sudo -S bazel-bin/tensorflow/tools/pip_package/build_pip_package ${WHL_DIR}
                '''
            }
    }
         stage('Install Tensorflow Package') {
            steps {
                  sh '''#!/bin/bash -xe
                  sudo -S pip --no-cache-dir install --upgrade ${WHL_DIR}/tensorflow-*.whl
                  sudo -S rm -rf /root/.cache
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
                   sudo -S md5sum /media/common/IT/${TFLOW} > ${TFLOW}.md5
                   sudo -S md5sum -c ${TFLOW}.md5 
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
