FROM nvidia/cuda:11.0-base
RUN apt-get update && apt-get  install -y x11vnc net-tools iputils-ping socat
ADD entrypoint /usr/bin/entrypoint
RUN chmod a+x /usr/bin/entrypoint
RUN apt-get install -y --reinstall libgtk2.0-0
COPY sudoers /etc/sudoers
RUN chown root:root /etc/sudoers && chmod a-rwx /etc/sudoers && chmod ug+r /etc/sudoers
CMD ["echo No command given!"]
ENTRYPOINT ["entrypoint"]
