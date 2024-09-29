# Basis image: ROS 2 Humble met desktop full
ARG BASE_IMAGE=humble-desktop-full

FROM osrf/ros:$BASE_IMAGE

MAINTAINER Gerard Harkema <ga.harkema@avans.nl>

ENV DEBIAN_FRONTEND=noninteractive

SHELL ["/bin/bash", "-c"]

# install depending packages
RUN apt-get update && apt-get upgrade -y && \
    apt-get install -y bash-completion \
	less \
	language-pack-en \
	nano \
	dos2unix\
	#code \
	#vim-tiny \
	#iputils-ping \
	net-tools \
	pip
	#\
	#openssh-client \
	#openjdk-8-jdk-headless \
	#nodejs \
	#imagemagick \
	#byzanz \
	#python-dev \
	#libsecret-1-dev && \
    #npm install -g yarn && \
RUN apt-get clean

# basic python packages
RUN pip install --upgrade --ignore-installed --no-cache-dir \
	pyassimp pylint==1.9.4 \
	autopep8 \
	python-language-server[all] \
	notebook~=5.7 \
	Pygments \
	matplotlib \
	ipywidgets \
	jupyter_contrib_nbextensions \
	nbimporter \
	supervisor \
	supervisor_twiddler \
	argcomplete 



# add non-root user
RUN useradd -m developer && \
    echo developer ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/developer && \
    chmod 0440 /etc/sudoers.d/developer

# install depending packages (install moveit! algorithms on the workspace side, since moveit-commander loads it from the workspace)


# Update en installeer vereiste pakketten
RUN apt-get install -y \
    git \
    curl \
    wget \
    lsb-release \
    python3-pip \
    software-properties-common \
    xfce4 \
    xfce4-goodies \
    x11vnc \
    xvfb \
    supervisor \
    dbus-x11 

# Configuratie voor VNC en supervisor
RUN mkdir -p /etc/supervisor/conf.d
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
RUN dos2unix /etc/supervisor/conf.d/supervisord.conf
#COPY code-server.conf /etc/supervisor/conf.d/code-server.conf

COPY entrypoint.sh /entrypoint.sh
RUN dos2unix /entrypoint.sh

#COPY --from=xsdcache /opt/xsd /opt/xsd

RUN apt-get install -y \
    ros-$ROS_DISTRO-moveit \
    ros-$ROS_DISTRO-moveit-ros-visualization \
    ros-$ROS_DISTRO-controller-manager-msgs \
    ros-$ROS_DISTRO-hardware-interface \
    ros-$ROS_DISTRO-joint-state-broadcaster \
    ros-$ROS_DISTRO-joint-trajectory-controller \
    ros-$ROS_DISTRO-joint-state-publisher \
    ros-$ROS_DISTRO-control-toolbox \
    ros-$ROS_DISTRO-controller-manager \
    ros-$ROS_DISTRO-desktop \
    ros-$ROS_DISTRO-moveit-servo 
#Add own ROS Packeges here

RUN curl -sSL http://get.gazebosim.org | sh

RUN apt-get install -y \
    ros-$ROS_DISTRO-gazebo-msgs \
    ros-$ROS_DISTRO-gazebo-ros2-control \
    ros-$ROS_DISTRO-gazebo-plugins \
    ros-$ROS_DISTRO-gazebo-ros 

# Uncoment for using UR Robots
#RUN apt-get install -y ros-$ROS_DISTRO-ur


RUN rm -rf /var/lib/apt/lists/*


USER developer
WORKDIR /home/developer

ENV HOME=/home/developer
ENV SHELL=/bin/bash

# enable bash completion
RUN git clone --depth=1 https://github.com/Bash-it/bash-it.git ~/.bash_it && \
    ~/.bash_it/install.sh --silent && \
    rm ~/.bashrc.bak && \
    echo "source /usr/share/bash-completion/bash_completion" >> ~/.bashrc && \
    bash -i -c "bash-it enable completion git"

RUN echo 'eval "$(register-python-argcomplete sim)"' >> ~/.bashrc

RUN git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf && \
    ~/.fzf/install --all

#RUN git clone --depth 1 https://github.com/b4b4r07/enhancd.git ~/.enhancd && \
#    echo "source ~/.enhancd/init.sh" >> ~/.bashrc

# colorize less
RUN lesspipe >> ~/.bashrc && \
    echo "export LESS='-R'" >> ~/.bashrc && \
    echo "export PYGMENTIZE_STYLE='monokai'" >> ~/.bashrc && \
    curl -sSL https://raw.githubusercontent.com/CoeJoder/lessfilter-pygmentize/master/.lessfilter > ~/.lessfilter && \
    chmod 755 ~/.lessfilter

# init rosdep
RUN rosdep update

# enter ROS world
RUN echo "source /opt/ros/$ROS_DISTRO/setup.bash" >> ~/.bashrc

EXPOSE 3000 8888

RUN mkdir -p /home/developer/installers
COPY installers /home/developer/installers
RUN sudo chmod -R 777 /home/developer/installers


RUN sudo chmod +x /entrypoint.sh
ENTRYPOINT [ "/entrypoint.sh" ]
CMD [ "sudo", "-E", "/usr/local/bin/supervisord", "-c", "/etc/supervisor/supervisord.conf"]
