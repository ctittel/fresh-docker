# docker-fun

## Build commands
### ws-bridge
- `docker build --tag ctonic/ws-bridge https://github.com/ctonic/fresh-docker.git#:ws-bridge`
- `docker run -dit --rm --gpus '"device=0"' -p 9090:9090 --name c_ws_bridge ctonic/ws-bridge`

### simulator
- `docker build -t ctonic/nvidia-vnc https://github.com/ctonic/fresh-docker.git#:simulator`
- `docker run -v /home/christoph/data/simulator:/data -p 9090:9090 -p 5900:5900 --gpus '"device=0"' -it --name c_simulator ctonic/nvidia-vnc ./data/ManipulatorEnvironment_Linux.x86_64`
