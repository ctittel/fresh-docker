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
    - configure the environment variables in `settings` depending on your local setup
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
            -e DISPLAY \
            -v /tmp/.X11-unix:/tmp/.X11-unix:rw \
            -v "$agent_path":/agent:rw \
            -u $(id -u $USER):$(id -g $USER) \
            ctonic/agent \
            bash -c "cd agent && $agent_start"
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
            -e DISPLAY \
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
            bash -c "cd  /home/$USER/simulation/ && $simulation_start"
```

**Note**: The unity simulation program must be able to write in `~/.config`. If there are problems with e.g. the screen resolution, delete this folder before starting the simulation again.

## How to run
**Important:** Before running any of the steps below you may need to customize and then source the environment variables of your bash shell in the `settings` file in this directory. Do this by running `source settings` when inside it.

**Important:** After running the commands below always disconnect your SSH connection gracefully, with `exit`. Otherwise the processes started below may also quit when you disconnect.

### Starting or finding a X11 display
If graphical applications are launched in the Docker containers, there must be a Xorg server they can reach.
This section explains how you can start such a Xorg server.

First check if there is already a Xorg server running.
To do this, run `ps aux | grep X`.

An output line like this indicates that we have displays `:3` and `:5` available:
```
root     24401  0.7  0.1 292064 77724 tty2     Ssl+ 01:02   6:25 /usr/lib/xorg/Xorg :3 vt2
root     17669  1.0  0.0 272336 57404 tty3     Ssl+ 15:26   0:00 /usr/lib/xorg/Xorg :5
```

If you have no available displays or want to start a new one for other reasons,
you can start one with this command. Replace `$display` with a display of your choice, e.g. `:3`:
```bash
xinit `which bash` -- $display &
```

Next, you need to set the display variable.
Here we will use display `:3`.
This depends on which GPU you want to use.
Gui programs send their OpenGL instructions to the GPU of the display they are attached to.
If you have two GPUs and want to run your GUI program on the second one, set the `$DISPLAY` variable to `:3.1`.

Examples:
- `export DISPLAY=:3`: Use the first GPU (equals `:3.0`)
- `export DISPLAY=:3.0`: Use the first GPU
- `export DISPLAY=:3.1`: Use the second GPU

Make sure the `$DISPLAY` variable is set with the display of your choice in every shell you use to run the following steps.

As a last step, make sure your chosen display has the correct virtual resolution by running:
```bash
xrandr --fb 1920x1200
```

### Starting the VNC server
Make sure your `$DISPLAY` variable is set correctly (see above). 
Then start the VNC server detached in the background by running:
```bash
nohup x11vnc -nopw -forever -shared -quiet &
```

Nohup will create a `nohup.out` file in the directory you are running it from.
If the VNC server is started successfully, the content of this file will look like this:
```
The VNC desktop is:      ai4di-i06se2:0
PORT=5900
```

### Connecting to the VNC server from your client
Run the following two commands on your client computer.
This assumes that the VNC server is running on its default port `5900` on the host.
If there are multiple VNC servers running on the host, they will use the ports `5901`, `5902`, ...
Establish SSH bridge to the VNC server: `ssh -N -T -L 5900:localhost:5900 user@remotehost &`
On your PC: Start the vnc client: `vncviewer localhost:5900`

### Running agent and simulation
If you simulation and/or agent are using a GUI, make sure that the `$DISPLAY` variable is set correctly in the shell you are using.
Then run the `./start_containers` bash script in this repository. 

# Useful links
- http://wiki.ros.org/docker/Tutorials/GUI
