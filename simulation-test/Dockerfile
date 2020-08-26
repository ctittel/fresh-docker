FROM nvidia/cuda:11.0-base
RUN apt-get update && apt-get  install -y x11vnc net-tools iputils-ping socat
WORKDIR /root
RUN apt-get update && apt-get  install -y wget xserver-xorg-video-dummy
RUN apt-get install -y libglu1-mesa
# for VirtualGL
RUN wget https://sourceforge.net/projects/virtualgl/files/2.6.4/virtualgl_2.6.4_amd64.deb &&\
    dpkg -i virtualgl_2.6.4_amd64.deb

# remove unused files
RUN rm -rf /var/lib/apt/lists/* &&\
    rm virtualgl_2.6.4_amd64.deb

ADD entrypoint /usr/bin/entrypoint
RUN chmod a+x /usr/bin/entrypoint
# RUN apt-get install -y --reinstall libgtk2.0-0
# CMD ["echo No command given!"]
# ENTRYPOINT ["entrypoint"]
