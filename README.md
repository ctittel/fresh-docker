# docker-fun

Used ports:
- 9090 for ROS bridge (can be done within a network)
- 22 for SSH (can be done within a network)
- 5900 for VNC (must be published to host)

## Rund and Build commands
### ws-bridge
- `docker build --tag ctonic/ws-bridge https://github.com/ctonic/fresh-docker.git#:ws-bridge`
- `docker run --rm --network c_network -dit --gpus '"device=0"' -v /home/christoph/data/ssh:/root/.ssh --name c_ws_bridge ctonic/ws-bridge`

### simulator
- `docker build -t ctonic/nvidia-vnc https://github.com/ctonic/fresh-docker.git#:simulator`
- `docker run --rm --network c_network -dit --gpus '"device=0"' -v /home/christoph/data/ssh:/root/.ssh -v /home/christoph/data/simulator:/data -p 5900:5900 --name c_simulator ctonic/nvidia-vnc "bash" "-c" "sleep 5 && ./data/ManipulatorEnvironment_Linux.x86_64"`
