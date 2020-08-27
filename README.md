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
```bash
docker build -t ctonic/agent \
        https://github.com/ctonic/fresh-docker.git#:agent
```

Running:
```bash
docker run  --rm -it \
            --network c_network \
            --gpus '"device=0"' \
            -v /home/christoph/data/agent:/agent \
            --name c_agent \
            -u $(id -u $USER):$(id -g $USER) \
            ctonic/agent \
            bash -c "cd agent && python main.py"
```

### agent-vnc
Builds on top of the agent container and also contains vnc applications.
This is useful if you want to see the OpenAi window via vnc (for training and viewing the Unity simulation use `agent` and `simulation`).

Building: 
```bash
docker build -t ctonic/agent-vnc \
        https://github.com/ctonic/fresh-docker.git#:agent-vnc
```

Running:
```bash
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
```bash
docker build --tag ctonic/ros-bridge \
https://github.com/ctonic/fresh-docker.git#:ros-bridge
```

Running: 
```bash
docker run  --rm -it \
            --network c_network \
            -p 9090:9090 \
            --gpus '"device=0"' \
            --name c_ros_bridge \
            ctonic/ros-bridge
```

### Simulation
Running:
```bash
docker run  --rm -it \
            --name c_simulation \
            --net c_network \
            -u $(id -u $USER):$(id -g $USER) \
            -e DISPLAY \
            -v /tmp/.X11-unix:/tmp/.X11-unix:rw \
            -v  /etc/group:/etc/group:ro \
            -v /etc/passwd:/etc/passwd:ro \
            -v /etc/shadow:/etc/shadow:ro \
            -v /home/christoph:/home/christoph \
            -v /etc/sudoers.d:/etc/sudoers.d:ro \
            --privileged \
            --env="QT_X11_NO_MITSHM=1" \
            --gpus '"device=0"' \
            nvidia/cudagl:10.0-devel-ubuntu18.04 \
            bash -c "cd home/christoph/data/simulation && ./ManipulatorEnvironment_Linux.x86_64"
```



## How to run
### Running acrobot_unity, ManipulatorEnvironment and VNC
- Preparation:
    - Build `agent` with the given build command above
1. Start a new X11 instance on the server (if there isn't one already): 
```bash
xinit `which bash` -- :3 vt2
```
2. Change its screen resolution if you want: `DISPLAY=:3 xrandr --fb 1600x900`
3. Optional: Start the vnc server and connect to it with your PC
    1. Start VNC server: `DISPLAY=:3 x11vnc -nopw -forever -shared`
    2. On your PC: Establish SSH bridge to the VNC server: `ssh -N -T -L 5900:localhost:5900 user@remotehost &`
    3. On your PC: Start the vnc client: `vncviewer localhost:5900`
4. Start `simulation`:
    1. Set the `DISPLAY` variable in your current SSH shell so the simulation knows its X server: `export DISPLAY=:3`
    2. Start `simulation` with the corresponding `docker run` command above
5. Start `agent` with the corresponding `docker run` command above

### Running agent for training with OpenAI gym
Execute the two commands given under `agent-vnc`.
To connect to it with your PC:
Execute `ssh -N -T -L 5900:localhost:5900 user@remotehost &` and `vncviewer localhost:5900` on your PC.

# Useful links
- http://wiki.ros.org/docker/Tutorials/GUI
