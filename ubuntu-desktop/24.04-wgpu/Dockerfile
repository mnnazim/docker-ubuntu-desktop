ARG BASE_IMAGE=ubuntu:24.04
FROM $BASE_IMAGE


ARG NOVNC_VERSION=1.4.0
ARG CHROME_VERSION=131.0.6778.204
LABEL maintainer "nazim324"

ENV DEBIAN_FRONTEND=noninteractive
ENV USER=ubuntu \
    PASSWORD=ubuntu \
    UID=1000 \
    GID=1000

ENV NVIDIA_VISIBLE_DEVICES=all
ENV NVIDIA_DRIVER_CAPABILITIES=all
ENV HTTPS_CERT=/etc/ssl/certs/ssl-cert-snakeoil.pem
ENV HTTPS_CERT_KEY=/etc/ssl/private/ssl-cert-snakeoil.key
ENV VGL_DISPLAY=egl
ENV REMOTE_DESKTOP=novnc
ENV VNC_THREADS=2


## Copy config
COPY docker_config /docker_config

RUN chmod -R +x /docker_config


#mount volume
RUN mkdir -p /home/ubuntu/data &&\
    chown -R ubuntu:ubuntu /home/ubuntu/data

#VOLUME mwdata:/home/ubuntu/data

RUN cp /docker_config/config.json /home/ubuntu/data/config.json
# Remove default user ubuntu if exist
RUN userdel ubuntu || true

## Install and Configure OpenGL
## Install and Configure Vulkan

## Install some common tools 
RUN apt update && \
    apt install -y sudo vim nano gedit locales gnupg2 wget curl zip lsb-release bash-completion fonts-beng &&\
    apt install -y net-tools iputils-ping mesa-utils software-properties-common build-essential &&\
    apt install -y python3 python3-pip python3-numpy &&\
    python3 -m pip config set global.break-system-packages true && \
    apt install -y openssh-server openssl git git-lfs tmux &&\
    rm -rf /var/lib/apt/lists/* 

## Install desktop
RUN apt-get update && \
    # add apt repo for firefox
    #add-apt-repository -y ppa:mozillateam/ppa &&\
    #mkdir -p /etc/apt/preferences.d &&\
    #echo "Package: firefox*\n\
#Pin: release o=LP-PPA-mozillateam\n\
#Pin-Priority: 1001" > /etc/apt/preferences.d/mozilla-firefox &&\
    # install xfce4 and firefox
    apt-get install -y xfce4 terminator fonts-wqy-zenhei ffmpeg dbus-x11 &&\
    # remove and disable screensaver
    apt-get remove -y xfce4-screensaver --purge &&\
    # set firefox as default web browser
    #update-alternatives --set x-www-browser /usr/bin/firefox &&\
    rm -rf /var/lib/apt/lists/*



## Install Chromium
RUN apt-get update &&\
    echo $CHROME_VERSION && \
    wget https://dl.google.com/linux/chrome/deb/pool/main/g/google-chrome-stable/google-chrome-stable_$CHROME_VERSION-1_amd64.deb  &&\
    apt-get install -y ./google-chrome-stable_$CHROME_VERSION-1_amd64.deb &&\
    rm google-chrome-stable_$CHROME_VERSION-1_amd64.deb &&\
    update-alternatives --set x-www-browser /usr/bin/google-chrome-stable &&\
    wget https://storage.googleapis.com/chrome-for-testing-public/$CHROME_VERSION/linux64/chromedriver-linux64.zip &&\
    unzip chromedriver-linux64.zip &&\
    mv chromedriver-linux64/chromedriver /usr/bin/chromedriver &&\
    rm chromedriver-linux64.zip &&\
    rm -r chromedriver-linux64 && \
    rm -rf /var/lib/apt/lists/*


ENV DBUS_SYSTEM_BUS_ADDRESS=unix:path=/host/run/dbus/system_bus_socket
RUN apt-get update && apt-get install -y pulseaudio && mkdir -p /var/run/dbus &&\
    rm -rf /var/lib/apt/lists/*

## Configure ssh 
RUN mkdir -p /var/run/sshd && \
    sed -i 's/#*PermitRootLogin prohibit-password/PermitRootLogin yes/g' /etc/ssh/sshd_config && \
    sed -i 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' /etc/pam.d/sshd

## Install remote desktop and other apps
RUN wget -q -O- https://packagecloud.io/dcommander/turbovnc/gpgkey | gpg --dearmor >/etc/apt/trusted.gpg.d/TurboVNC.gpg &&\
    wget -O /etc/apt/sources.list.d/TurboVNC.list https://raw.githubusercontent.com/TurboVNC/repo/main/TurboVNC.list &&\
    apt-get update &&\
    apt-get install -y turbovnc  &&\
    rm /etc/apt/sources.list.d/TurboVNC.list  &&\
    echo "installing novnc...$NOVNC_VERSION" &&\
    # install novnc
    curl -fsSL "https://github.com/novnc/noVNC/archive/refs/tags/v1.4.0.tar.gz" | tar -xzf - -C /opt  &&\
    mv -f "/opt/noVNC-1.4.0" /opt/noVNC  &&\
    ln -snf /opt/noVNC/vnc.html /opt/noVNC/index.html  &&\
    git clone "https://github.com/novnc/websockify.git" /opt/noVNC/utils/websockify  &&\
    # config turbovnc
    echo "xset s off && /usr/bin/startxfce4" > /opt/TurboVNC/bin/xstartup.turbovnc  &&\
    rm -rf /var/lib/apt/lists/*

RUN apt-get update && \
    apt-get install -yq tzdata && \
    #ln -fs /usr/share/zoneinfo/America/New_York /etc/localtime && \
    #dpkg-reconfigure -f noninteractive tzdata
    rm -rf /var/lib/apt/lists/*

ENV TZ="Asia/Dhaka"


RUN pip install --no-cache-dir -r /docker_config/requirements.txt

RUN mkdir -p /home/ubuntu/.local/share/undetected_chromedriver &&\
    cp /usr/bin/chromedriver /home/ubuntu/.local/share/undetected_chromedriver/


EXPOSE 22 4000

ENTRYPOINT ["/docker_config/entrypoint.sh"]
