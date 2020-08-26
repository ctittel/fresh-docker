FROM dtur3/r3-ws-bridge
RUN apt-key adv --keyserver keyserver.ubuntu.com --recv-keys F42ED6FBAB17C654 && apt update && apt install -y openssh-server && mkdir /var/run/sshd && /usr/sbin/sshd
ADD entrypoint /usr/bin/entrypoint
RUN chmod a+x /usr/bin/entrypoint
ENTRYPOINT entrypoint