ARG VARIANT="17-bullseye"
FROM mcr.microsoft.com/vscode/devcontainers/java:${VARIANT}

ARG USERNAME=vscode
ARG NOVNC_VERSION=1.2.0
ARG WEBSOCKIFY_VERSION=0.9.0
ENV DEBIAN_FRONTEND noninteractive

ARG WIDHT=1024
ARG HEIGHT=768

ENV DISPLAY :0
ENV LANG=en_US.UTF-8 \
    LANGUAGE=en_US.UTF-8 \
    LC_ALL=C.UTF-8 \
    DISPLAY_WIDTH=${WIDHT} \
    DISPLAY_HEIGHT=${HEIGHT}


ARG CLOJURE_VERSION=1.10.3
ARG CLOJURE_CLI_VERSION=1.10.3.1075
ARG LEININGEN_VERSION="stable"
ARG POLYLITH_VERSION="0.2.13-alpha"
ENV BOOT_VERSION=2.8.3
ENV BOOT_CLOJURE_VERSION=${CLOJURE_VERSION}

ARG INSTALL_CLOJURE_CLI="true"
ARG INSTALL_BOOT="true"
ARG INSTALL_LEININGEN="true"
ARG INSTALL_POLYLITH="true"


RUN if [ "${INSTALL_CLOJURE_CLI}" = "true" ]; then \
    apt-get update \
    && apt-get -y install rlwrap \
    && curl -OL "https://download.clojure.org/install/linux-install-${CLOJURE_CLI_VERSION}.sh" \
    && chmod +x linux-install-${CLOJURE_CLI_VERSION}.sh \
    && /linux-install-${CLOJURE_CLI_VERSION}.sh \
    && rm /linux-install-${CLOJURE_CLI_VERSION}.sh \
    && su vscode -c "clj --version"; fi

RUN if [ "${INSTALL_BOOT}" = "true" ]; then \
    curl -OL "https://github.com/boot-clj/boot-bin/releases/download/latest/boot.sh" \
    && chmod +x boot.sh \
    && mv boot.sh /usr/local/sbin/boot \
    && su vscode -c "boot -u"; fi

RUN if [ "${INSTALL_LEININGEN}" = "true" ]; then \
    curl -OL "https://raw.githubusercontent.com/technomancy/leiningen/${LEININGEN_VERSION}/bin/lein" \
    && chmod +x lein \
    && mv lein /usr/local/sbin; fi

RUN if [ "${INSTALL_LEININGEN}" = "true" ]; then \
    su vscode -c " cd ~ \
    && echo '(defproject dummy \"\" :dependencies [[org.clojure/clojure \"'${CLOJURE_VERSION}'\"]])' > project.clj \
    && lein deps \
    && rm project.clj"; fi

RUN if [ "${INSTALL_POLYLITH}" = "true" ]; then \
    curl -OL "https://github.com/polyfy/polylith/releases/download/v${POLYLITH_VERSION}/poly-${POLYLITH_VERSION}.jar" \
    && mkdir -p /usr/local/polylith \
    && mv poly-$POLYLITH_VERSION.jar /usr/local/polylith \
    && echo '#!/bin/sh\nARGS=""\nwhile [ "$1" != "" ] ; do\n  ARGS="$ARGS $1"\n  shift\ndone\nexec "java" $JVM_OPTS "-jar" "/usr/local/polylith/poly-'$POLYLITH_VERSION'.jar" $ARGS\n' > /usr/local/sbin/poly \
    && chmod +x /usr/local/sbin/poly \
    && /usr/local/sbin/poly version; fi

RUN curl -sLO https://raw.githubusercontent.com/greglook/cljstyle/main/script/install-cljstyle \
    && chmod +x install-cljstyle \
    && ./install-cljstyle

ARG NODE_VERSION="lts/*"
RUN if [ "${NODE_VERSION}" != "none" ]; then su vscode -c "umask 0002 && . /usr/local/share/nvm/nvm.sh && nvm install ${NODE_VERSION} 2>&1"; fi


RUN apt-get install -y --no-install-recommends build-essential manpages-dev \
    git unzip dosfstools wget vim sudo ssh tzdata ca-certificates \
    xauth uuid-dev \
    dbus dbus-x11 alsa-utils pulseaudio pulseaudio-utils mesa-utils x11-apps \
    python3 python-is-python3 python3-distutils python3-numpy python3-pip python3-tk \
    xvfb x11vnc xdotool wget tar supervisor net-tools lxde xterm \
    && apt-get autoremove -y && apt-get clean -y && rm -rf /var/lib/apt/lists/*

# Install noVNC
RUN mkdir -p /usr/local/novnc \
    && curl -k -L -o /tmp/novnc.zip https://github.com/novnc/noVNC/archive/v${NOVNC_VERSION}.zip \
    && unzip /tmp/novnc.zip -d /usr/local/novnc/ \
    && cp /usr/local/novnc/noVNC-${NOVNC_VERSION}/vnc.html /usr/local/novnc/noVNC-${NOVNC_VERSION}/index.html \
    && curl -k -L -o /tmp/websockify.zip https://github.com/novnc/websockify/archive/v${WEBSOCKIFY_VERSION}.zip \
    && unzip /tmp/websockify.zip -d /usr/local/novnc/ \
    && ln -sf /usr/local/novnc/websockify-${WEBSOCKIFY_VERSION} /usr/local/novnc/noVNC-${NOVNC_VERSION}/utils/websockify \
    && rm -rf /tmp/novnc.zip /tmp/websockify.zip

RUN cd /usr/bin && sudo mv lxpolkit lxpolkit.bak

ADD supervisord.conf /etc/supervisor/conf.d/supervisord.conf


USER ${USERNAME}

# Install Rust
# ENV RUST_VERSION stable
# RUN curl https://sh.rustup.rs -sSf | sh -s -- -y --default-toolchain ${RUST_VERSION}
# ENV PATH $PATH:/home/${USERNAME}/.cargo/bin
# RUN rustup install stable
# RUN rustup install nightly
# RUN rustup component add rust-analysis rust-src rustfmt clippy
# RUN cargo install cargo-watch
# RUN cargo install cargo-check

RUN mkdir -p /home/${USERNAME}/workspace
WORKDIR /home/${USERNAME}/workspace

USER root
COPY ./.inject_bashrc /root
RUN cat /root/.inject_bashrc >> /home/${USERNAME}/.bashrc

EXPOSE 8085
CMD ["/usr/bin/supervisord"]

