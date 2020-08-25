FROM dtur3/r3-ws-bridge
ADD start_ws /usr/bin/start_ws
RUN chmod a+x /usr/bin/start_ws
RUN apt update && apt install -y openssh-server && mkdir /var/run/sshd && /usr/sbin/sshd
ENTRYPOINT start_ws
