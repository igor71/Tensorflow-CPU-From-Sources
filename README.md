# Tensorflow-CPU-From-Sources
Build tensorflow-cpu package from the sources on docker slave

Automativ Build Configured In Pipeline Build-Tensorflow-Package Jenkins Job

### Tensorflow-CPU-From-Sources Manual Build Steps:
```
 pv /media/common/DOCKER_IMAGES/TFlow-Build/yi-tflow-build-cpu.tar | docker load
 
 docker images  -->> note image ID
 
 docker tag <Image ID> yi/tflow:0.0
  
 docker run -it --name tflow_build -v /media:/media yi/tflow:0.0 /bin/bash 
 
 git clone --branch=master --depth=1 https://github.com/igor71/Tensorflow-GPU-From-Sources
 
 cd Tensorflow-GPU-From-Sources
 
 /bin/bash tflow-build.sh
 
 cd /whl
 
 TFLOW=$(ls | sort -V | tail -n 1)
 
 pv /whl/$TFLOW > /media/common/DOCKER_IMAGES/Tensorflow/Current/$TFLOW
 
 pip --no-cache-dir install --upgrade $TFLOW
 
 cd /
 
 ipython gpu_tf_check.py
 ```
