FROM nvidia/cudagl:10.0-devel-ubuntu18.04
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get  install -y ffmpeg libsm6 libxext6 python3.6 python3-pip
RUN python3 -m pip install --upgrade pip
ADD requirements.txt /home/requirements.txt
RUN pip install -r /home/requirements.txt