FROM nvidia/cuda:11.0-base
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get  install -y xorg x11vnc net-tools iputils-ping socat  # long download and important
COPY startx /usr/bin/startx
COPY xorg.conf /etc/X11/xorg.conf
RUN apt-get install -y xserver-xorg-video-dummy fluxbox ssh && chmod +x /usr/bin/startx                # may change

# ADD entrypoint /usr/bin/entrypoint
# RUN chmod a+x /usr/bin/entrypoint
# CMD ["echo No command given!"]
# ENTRYPOINT ["entrypoint"]


# docker run --network c_network -it --gpus '"device=0"' -v /home/christoph/data/simulation:/simulation -p 5900:5900 --name c_simulation  nvidia/cuda:11.0-base "bash"
# apt update
# apt install xorg openbox (openbox is window manager like fluxbox)
# asks for country

# Links:
# http://xpra.org/trac/wiki/Xdummy
# https://github.com/didzis/nvidia-xorg-virtualgl-docker
# very good: https://medium.com/@chadrick.kwag/configuring-x-server-on-headless-server-a8b5c3bc7d9