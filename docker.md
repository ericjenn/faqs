# Docker for ROS2
To use ROS2 in a docker container:
```
docker run -it --privileged  -v Work:/home/ubuntu/work --env="DISPLAY=192.168.0.1:0.0" --env="QT_X11_NO_MITSHM=1" --env="XAUTHORITY=$XAUTH" --volume="$XAUTH:$XAUTH" --net=host osrf/ros:jazzy-desktop-full /bin/bash
```
After starting the container, I add user "ubuntu" to the sudo group:
```
usermod -aG sudo ubuntu
```
Then I `su` to user "ubuntu".

I could have used option `--user 1000:1000` but, in that case, I can't use `sudo`because user `ubuntu`has no password.

The appropriate solution would be to create a dockerfile setup correctly...

