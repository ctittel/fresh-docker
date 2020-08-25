# docker-fun

Used ports:
- 9090 for ROS bridge (can be done within a network)
- 22 for SSH (can be done within a network)
- 5900 for VNC (must be published to host)

## VNC client instructions
Instructions to see the GUI via VNC on your client:
Run these commands on your clients if a vnc server (from `agent-vnc` or `simulation`) is running:

- Forward the VNC port from the server to your client: `ssh -N -T -L 5900:localhost:5900 user@remotehost &`
- Run the VNC client: `vncviewer localhost:5900`

## Assumptions
These are the ports, names and settings that are used in the given commands.
If you use other ports / names / settings, you have to substitute in the commands below.

### Ports
- VNC: 5900 (transfered to host server)
- other networking happening is SSH (port 22) and the ROS bridge (9090), but this happens in a self-contained network

### Container / Variable names
- `c_network`: self-contained Docker network
- `c_agent`: container running agent code (does the training)
- `c_ws_bridge`: container with the ROS bridge
- `c_simulation`: Runs the Unity simulation

### Settings
- Using the first GPU of the server, GPU 0
- In my home drive I have a directory called `/home/christoph/data` with the following subdirectories:
    - `agent`: contains all the agent code; `main.py` is inside this directory
    - `ssh`: contains a public and private key pair called `id_rsa` and `id_rsa.pub` (can be generated with `ssh-keygen`)
    - `simulation`: contains the unity application; the executable is directly below this (whole path to it: `data/simulation/ManipulatorEnvironment_Linux.x86_64`)

## Containers and Images
### agent
Contains NVIDIA drivers, tensorflow 1, torch and basically everything to do the training of OpenAi-Gym and the Unity simulation. 

- Building: `docker build -t ctonic/agent https://github.com/ctonic/fresh-docker.git#:agent`
- Running: `docker run --rm --network c_network -it --gpus '"device=0"' -v /home/christoph/data/agent:/agent --name c_agent ctonic/agent bash -c "cd agent && python main.py"`

### agent-vnc
Builds on top of the agent container and also contains vnc applications.
This is useful if you want to see the OpenAi window via vnc (for training and viewing the Unity simulation use `agent` and `simulation`).

- Building: `docker build -t ctonic/agent https://github.com/ctonic/fresh-docker.git#:agent-vnc`
- Running: `docker run --rm --network c_network -it -p 5900:5900 --gpus '"device=0"' -v /home/christoph/data/agent:/agent --name c_agent ctonic/agent bash -c "cd agent && python main.py"`

### ws-bridge
When training the Unity robot this must be run first.
Contains a `roscore` instance and a ROS websocket bridge for `agent` (when using the `acrobot-unity` environment) and `simulation` to connect to.

- Building: `docker build --tag ctonic/ws-bridge https://github.com/ctonic/fresh-docker.git#:ws-bridge`
- Running: `docker run --rm --network c_network -dit --gpus '"device=0"' -v /home/christoph/data/ssh:/ssh --name c_ws_bridge ctonic/ws-bridge`

### simulation
Runs the simulation and a VNC server. You can connect to the vnc with the instructions above.

- Building: `docker build -t ctonic/simulation https://github.com/ctonic/fresh-docker.git#:simulation`
- Running: `docker run --rm --network c_network -dit --gpus '"device=0"' -v /home/christoph/data/ssh:/ssh -v /home/christoph/data/simulation:/simulation -p 5900:5900 --name c_simulation ctonic/simulation "bash" "-c" "sleep 5 && ./simulation/ManipulatorEnvironment_Linux.x86_64"`

## How to run

- Adapt the commands above with your settings / naming / ports etc.
- Build the required images
    - For `acrobot-unity`: `agent`, `simulation` and `ws-bridge`
    - For OpenAi Gym: `agent-vnc`
- Run in the correct order, for `acrobot-unity`:
    1. Run `c_ws_bridge`
    2. Run `c_simulation`
    3. Optional: Connect to `c_simulation` via VNC from you client
    4. Run `agent`