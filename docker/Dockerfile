FROM nvidia/cuda:10.0-cudnn7-runtime-ubuntu18.04

##################
## Requirements ##
##################
RUN apt-get update && apt-get install -y git wget && \
rm -rf /var/lib/apt/lists/*
RUN export LD_LIBRARY_PATH=/usr/local/cuda/lib64

###############
## NMT-Keras ##
###############
RUN git clone https://github.com/PRHLT/nmt-keras_practicas-TA.git /opt/nmt-keras \
&& cp /opt/nmt-keras/docker/config.py /opt/nmt-keras/config.py
WORKDIR /opt/nmt-keras
RUN wget https://repo.continuum.io/miniconda/Miniconda3-py37_4.8.3-Linux-x86_64.sh -O /tmp/miniconda.sh && \
bash /tmp/miniconda.sh -b -p /opt/miniconda && export PATH="/opt/miniconda/bin:$PATH" && \
conda config --set always_yes yes --set changeps1 no && conda init && \
conda install --file /opt/nmt-keras/req-travis-conda.txt && conda install mkl mkl-service pip tensorflow-gpu=1.14 && \
pip install -r /opt/nmt-keras/req-travis-pip.txt && pip install tensorflow==1.14.0
ENV PYTHONPATH="PYTHONPATH=${PYTHONPATH}:/opt/nmt-keras/keras:\
/opt/nmt-keras/coco-caption:/opt/nmt-keras/multimodal_keras_wrapper"

##########################
## Default: train model ##
##########################
CMD export PATH="/opt/miniconda/bin:$PATH" && \
export LD_LIBRARY_PATH=/usr/local/cuda/lib64 && \
python main.py
