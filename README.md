# docker-fun

Used ports:
- 9090 for ROS bridge (can be done within a network)
- 22 for SSH (can be done within a network)
- 5900 for VNC (must be published to host)

## Rund and Build commands
### ws-bridge
- `docker build --tag ctonic/ws-bridge https://github.com/ctonic/fresh-docker.git#:ws-bridge`
- `docker run --rm --network c_network -dit --gpus '"device=0"' -v /home/christoph/data/ssh:/ssh --name c_ws_bridge ctonic/ws-bridge`

### simulator
- `docker build -t ctonic/simulator https://github.com/ctonic/fresh-docker.git#:simulator`
- `docker run --rm --network c_network -dit --gpus '"device=0"' -v /home/christoph/data/ssh:/ssh -v /home/christoph/data/simulator:/simulator -p 5900:5900 --name c_simulator ctonic/simulator "bash" "-c" "sleep 5 && ./simulator/ManipulatorEnvironment_Linux.x86_64"`

### agent
- `docker build -t ctonic/agent https://github.com/ctonic/fresh-docker.git#:agent`
- `docker run --rm --network c_network -dit --gpus '"device=0"' -v /home/christoph/data/agent:/agent --name c_agent ctonic/agent python agent/main.py`