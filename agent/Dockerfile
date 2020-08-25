FROM tensorflow/tensorflow:1.15.2-gpu
RUN apt-get update && apt-get  install -y ffmpeg libsm6 libxext6
ADD requirements.txt /home/requirements.txt
RUN pip install -r /home/requirements.txt
