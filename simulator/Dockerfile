FROM nvidia/cuda:11.0-base
RUN apt-get update && apt-get  install -y x11vnc xvfb fluxbox net-tools iputils-ping ssh
ADD start_vnc /usr/bin/start_vnc
RUN chmod a+x /usr/bin/start_vnc
CMD ["echo No command given!"]
ENTRYPOINT ["start_vnc"]
