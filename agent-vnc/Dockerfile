FROM ctonic/agent
RUN apt-get update && apt-get  install -y x11vnc xvfb fluxbox python-opengl
ADD entrypoint /usr/bin/entrypoint
RUN chmod a+x /usr/bin/entrypoint
CMD ["echo No command given!"]
ENTRYPOINT ["entrypoint"]