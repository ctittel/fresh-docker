FROM nvidia/cuda:11.0-base
RUN apt-get update && apt-get  install -y x11vnc xvfb fluxbox net-tools iputils-ping socat
ADD entrypoint /usr/bin/entrypoint
RUN chmod a+x /usr/bin/entrypoint
CMD ["echo No command given!"]
ENTRYPOINT ["entrypoint"]
