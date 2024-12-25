# Docker for ROS2
To use ROS2 in a docker container:
```
docker run -it --privileged  -v Work:/home/ubuntu/work --env="DISPLAY=192.168.0.1:0.0" --user 1000:1000 --env="QT_X11_NO_MITSHM=1" --env="XAUTHORITY=$XAUTH" --volume="$XAUTH:$XAUTH" --net=host osrf/ros:jazzy-desktop-full /bin/bash
```

