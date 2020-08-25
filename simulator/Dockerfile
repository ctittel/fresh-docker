FROM nvidia/cuda:11.0-base
RUN apt update && apt  install -y x11vnc xvfb fluxbox net-tools
ADD start_vnc /usr/bin/start_vnc
RUN chmod a+x /usr/bin/start_vnc
CMD ["echo No command given!"]
ENTRYPOINT ["start_vnc"]
