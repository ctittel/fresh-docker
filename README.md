# fresh-docker
## Setup
- On the server:
    - install docker
    - install `xorg`
    - install `x11vnc`
    - create the `xorg.conf` configuration file for the x11 server
        - on a headless server with nvidia GPUs, this can be done with this command: `sudo nvidia-xconfig -a --use-display-device=None`
        - see also: https://carla.readthedocs.io/en/0.9.7/carla_headless/
    - edit the file `/etc/X11/Xwrapper.config` to include the line `allowed_users=anybody`
        - this is required to be able to start a x11 server from the command line via SSH
    - clone this repository
    - cd into this repository and run `chmod a+x install && ./install`
- On your client:
    - install a vnc client, e.g. `tigervnc`

## Containers and Images
The build commands below use relative references to the `Dockerfile`'s.
Enter a local clone of this repository (e.g. with `cd fresh-docker`) before executing them.
The run commands contain environment variables which can be set with `source settings` when inside `fresh-docker`.

### agent
Contains NVIDIA drivers, tensorflow 1, torch and basically everything to do the training of OpenAi-Gym and the Unity simulation. 

Building:
```bash
docker build -t ctonic/agent agent
```

Running:
```bash
docker run  --rm -it \
            --name "$name_agent" \
            --net "$network" \
            --gpus "$gpus" \
            -v "$agent_path":/agent:rw \
            -u $(id -u $USER):$(id -g $USER) \
            ctonic/agent \
            bash -c "cd agent && $agent_start"
```

### agent-vnc
Builds on top of the agent container and also contains vnc applications.
This is useful if you want to see the OpenAi window via vnc (for training and viewing the Unity simulation use `agent` and `simulation`).

Building: 
```bash
docker build -t ctonic/agent-vnc agent-vnc
```

Running:
```bash
docker run  --rm -it \
            --name "$name_agent" \
            --net "$network" \
            --gpus "$gpus" \
            -p 5900:5900 \
            -v /home/christoph/data/agent:/agent:rw \
            ctonic/agent-vnc \
            bash -c "cd agent && python main.py"
```

### ros-bridge
When training the Unity robot this must be run first.
Contains a `roscore` instance and a ROS websocket bridge for `agent` (when using the `acrobot-unity` environment) and `simulation` to connect to.

Building:
```bash
docker build --tag ctonic/ros-bridge ros-bridge
```

Running: 
```bash
docker run  --rm -it \
            --name "$name_bridge" \
            --net $network \
            --gpus "$gpus" \
            -p 9090:9090 \
            ctonic/ros-bridge
```

### Simulation
Running:
```bash
docker run  --rm -it \
            --name "$name_simulation" \
            --net "$network" \
            --gpus "$gpus" \
            -u $(id -u $USER):$(id -g $USER) \
            -e DISPLAY=$display \
            -v /tmp/.X11-unix:/tmp/.X11-unix:rw \
            -v  /etc/group:/etc/group:ro \
            -v /etc/passwd:/etc/passwd:ro \
            -v /etc/shadow:/etc/shadow:ro \
            -v "$simulation_path":/home/$USER/simulation:rw \
            -v /home/$USER/.config:/home/$USER/.config:rw \
            -v /etc/sudoers.d:/etc/sudoers.d:ro \
            --privileged \
            --env="QT_X11_NO_MITSHM=1" \
            nvidia/cudagl:10.0-devel-ubuntu18.04 \
            bash -c "cd  /home/christoph/simulation/ && $simulation_start"
```

**Note**: The unity simulation program must be able to write in `~/.config`. If there are problems with e.g. the screen resolution, delete this folder before starting the simulation again.


## How to run
### Running acrobot_unity, ManipulatorEnvironment and VNC
- Preparation:
    - Build `agent` with the given build command above
1. Start a new X11 instance on the server (if there isn't one already): 
```bash
xinit `which bash` -- $display vt2
```
2. Change its screen resolution if you want: `DISPLAY=$display xrandr --fb 1920x1200`
3. Optional: Start the vnc server and connect to it with your PC
    1. Start VNC server: `DISPLAY=$display x11vnc -nopw -forever -shared`
    2. On your PC: Establish SSH bridge to the VNC server: `ssh -N -T -L 5900:localhost:5900 user@remotehost &`
    3. On your PC: Start the vnc client: `vncviewer localhost:5900`
4. Start `agent` with the corresponding `docker run` command above
5. Start `simulation`:
    1. Set the `DISPLAY` variable in your current SSH shell so the simulation knows its X server: `export DISPLAY=:3`
    2. Start `simulation` with the corresponding `docker run` command above

### Running agent for training with OpenAI gym
Execute the two commands given under `agent-vnc`.
To connect to it with your PC:
Execute `ssh -N -T -L 5900:localhost:5900 user@remotehost &` and `vncviewer localhost:5900` on your PC.

# Useful links
- http://wiki.ros.org/docker/Tutorials/GUI
