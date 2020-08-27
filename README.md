# fresh-docker
## Assumptions
These are the ports, names and settings that are used in the given commands.
If you use other ports / names / settings, you have to substitute in the commands below.

### Ports
- the following ports are exposed to the host server, make sure they are free or choose different ones:
    - VNC server runs on port `5900`
    - ROS bridge uses port `9090`

### Container / Variable names
- `c_network`: self-contained Docker network
- `c_agent`: container running agent code (does the training)
- `c_ros_bridge`: container with the ROS bridge
- `c_simulation`: Runs the Unity simulation

### Settings
- Using the first GPU of the server, GPU 0
- In my home drive I have a directory called `/home/christoph/data` with the following subdirectories:
    - `agent`: contains all the agent code; `main.py` is inside this directory
    - `simulation`: contains the unity application; the executable is directly below this (whole path to it: `data/simulation/ManipulatorEnvironment_Linux.x86_64`)

## Containers and Images
### agent
Contains NVIDIA drivers, tensorflow 1, torch and basically everything to do the training of OpenAi-Gym and the Unity simulation. 

Building:
```sh
docker build -t ctonic/agent \
        https://github.com/ctonic/fresh-docker.git#:agent
```

Running:
```sh
docker run  --rm -it \
            --network c_network \
            --gpus '"device=0"' \
            -v /home/christoph/data/agent:/agent \
            --name c_agent \
            ctonic/agent \
            bash -c "cd agent && python main.py"
```

### agent-vnc
Builds on top of the agent container and also contains vnc applications.
This is useful if you want to see the OpenAi window via vnc (for training and viewing the Unity simulation use `agent` and `simulation`).

Building: 
```sh
docker build -t ctonic/agent-vnc \
        https://github.com/ctonic/fresh-docker.git#:agent-vnc
```

Running:
```sh
docker run  --rm -it \
            --network c_network \
            -p 5900:5900 \
            --gpus '"device=0"' \
            -v /home/christoph/data/agent:/agent \
            --name c_agent \
            ctonic/agent-vnc \
            bash -c "cd agent && python main.py"
```

### ros-bridge
When training the Unity robot this must be run first.
Contains a `roscore` instance and a ROS websocket bridge for `agent` (when using the `acrobot-unity` environment) and `simulation` to connect to.

Building:
```sh
docker build --tag ctonic/ros-bridge \
https://github.com/ctonic/fresh-docker.git#:ros-bridge
```

Running: 
```sh
docker run  --rm -it \
            --network c_network \
            -p 9090:9090 \
            --gpus '"device=0"' \
            --name c_ros_bridge \
            ctonic/ros-bridge
```

### Simulation
1. Start a new X11 instance on the server (if there isn't one already): **xinit `which bash` -- :3 vt2**
2. Change its screen resolution if you want: `DISPLAY=:3 xrandr --fb 1920x1080`
3. Start the vnc server: `DISPLAY=:3 x11vnc -nopw -forever -shared`
4. Start the simulation (after starting the ros-bridge): `DISPLAY=:3 ./ManipulatorEnvironment_Linux.x86_64`
5. On the client: Bridge the VNC server port from the server to your client `ssh -N -T -L 5900:localhost:5900 user@remotehost &` and start your vnc client `vncviewer localhost:5900`

## How to run

- Adapt the commands above with your settings / naming / ports etc.
- Build the required images
    - For `acrobot-unity`: `agent`, `simulation` and `ros-bridge`
    - For OpenAi Gym: `agent-vnc`
- Run in the correct order, for `acrobot-unity`:
    1. Run `c_ros_bridge`
    2. Run `c_simulation`
    3. Wait > 10 seconds
    4. Optional: Connect to `c_simulation` via VNC from you client; The client should show that it is connected to `RosBridge`
    5. Wait a few seconds
    6. Run `agent`

# Useful links
- http://wiki.ros.org/docker/Tutorials/GUI
