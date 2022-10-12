FROM nvidia/cuda:11.3.1-runtime-ubuntu18.04
ENV LANG=C.UTF-8

RUN apt-get update && apt-get install -y git vim tree curl unzip xvfb patchelf ffmpeg cmake swig wget
RUN apt-get update && apt-get install -y libssl-dev libcurl4-openssl-dev  # Needed for pyCurl
RUN apt-get update && apt-get install -y \
    libglu1-mesa-dev libgl1-mesa-dev libosmesa6-dev \
    cmake libopenmpi-dev python3-dev zlib1g-dev \
    qtbase5-dev qtdeclarative5-dev libglib2.0-0

RUN apt-get install -y python3.8-dev python3-pip
RUN ln -s /usr/bin/pip3 /usr/bin/pip
RUN ln -s /usr/bin/python3.8 /usr/bin/python
RUN python -m pip install --upgrade pip

RUN apt-get install -y libgl1-mesa-glx libosmesa6
RUN apt-get install -y libglew-dev
RUN apt-get install -y libglfw3 libglfw3-dev
# used for egl rendering
# ENV LD_PRELOAD /usr/lib/x86_64-linux-gnu/libGLEW.so


###### Install mujoco ######
RUN mkdir /usr/local/.mujoco && \
    cd /usr/local/.mujoco  && \
    curl -O https://www.roboti.us/download/mujoco200_linux.zip  && \
    unzip mujoco200_linux.zip
RUN wget https://roboti.us/file/mjkey.txt -P /usr/local/.mujoco/

ENV MUJOCO_GL=egl
ENV MUJOCO_DIR=/usr/local/.mujoco
ENV LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$MUJOCO_DIR/mujoco200_linux/bin
ENV MUJOCO_PY_MUJOCO_PATH=$MUJOCO_DIR/mujoco200_linux
ENV MUJOCO_PY_MJKEY_PATH=$MUJOCO_DIR/mjkey.txt
ENV MJLIB_PATH=$MUJOCO_DIR/mujoco200_linux/bin/libmujoco200.so
ENV MJKEY_PATH=${MUJOCO_PY_MJKEY_PATH}

###### dataset ######
# Download DAVIS dataset required for distracting-control
ADD https://data.vision.ee.ethz.ch/csergi/share/davis/DAVIS-2017-trainval-480p.zip /tmp/
RUN mkdir /data
RUN unzip /tmp/DAVIS-2017-trainval-480p.zip -d /data/dataset && rm /tmp/DAVIS-2017-trainval-480p.zip
ENV DC_BG_PATH=/data/dataset/DAVIS/JPEGImages/480p

# NOTE: This dataset is huge (26GB) and only necessary for SODA.
# We thus decided to launch SODA experiments on our cluster.
# Download Places dataset required for SODA
# ADD http://data.csail.mit.edu/places/places365/places365standard_easyformat.tar /tmp/
# RUN tar -xvf /tmp/places365standard_easyformat.tar -d /root/dataset && rm /tmp/places365standard_easyformat.tar
# ENV PLACES_DATASET_PATH /root/dataset/places365_standard

# ENV PIP_INSTALL_OPTION=--with-openssl  # NOTE: <-- This makes torch installation fail...
ENV PIP_NO_CACHE_DIR off
# ENV PYCURL_SSL_LIBRARY openssl  # Unfortunately this is necessary to prevent pycurl from complaining


###### pip packages ######
# Basic stuff
RUN pip install cmx opencv-python pandas functional-notations pyyaml scikit-image xmltodict google-api-python-client google-cloud-storage

# machine learning framework
RUN pip install torch==1.9.0+cu111 torchvision==0.10.0+cu111 torchaudio==0.9.0 --use-deprecated=html5lib --find-links https://download.pytorch.org/whl/torch_stable.html

RUN pip install Cython lockfile cffi glfw # mujoco-py dependency

# Latest commit assumes mujoco v2.1, whereas we use mujoco v2.0
RUN pip install absl-py  # dm_control dependency (build-time only??)
RUN git clone git://github.com/deepmind/dm_control.git && \
    cd dm_control && \
    git checkout ff8a9caac9d845c1e1f669c272294b3355d6d855 && \
    python setup.py install --headers-dir=$MUJOCO_PY_MUJOCO_PATH/include

# RUN pip install --headers-dir=$MUJOCO_PY_MUJOCO_PATH git+git://github.com/deepmind/dm_control.git@ff8a9caac9d845c1e1f669c272294b3355d6d855
RUN pip install distracting-control==0.1.1rc3
RUN pip install gym-dmc mujoco-py==2.0.2.5

# geyang
RUN pip install jaynes waterbear params-proto ml-logger

# dev dependencies
RUN pip install flask flask-cors flake8 pylint mypy ipython kornia ipdb

# NOTE: Singularity fails without this as mujoco writes a lockfile here (why this isn't a issue in other places??)
RUN chmod -R a+w /usr/local/lib/python3.8/dist-packages/mujoco_py/generated
