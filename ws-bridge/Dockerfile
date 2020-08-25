FROM dtur3/r3-ws-bridge
ADD start_ws /usr/bin/start_ws
RUN chmod a+x /usr/bin/start_ws
RUN apt-key adv --keyserver keyserver.ubuntu.com --recv-keys F42ED6FBAB17C654 && apt update && apt install -y openssh-server && mkdir /var/run/sshd && /usr/sbin/sshd
ENTRYPOINT entrypoint-ws-bridge