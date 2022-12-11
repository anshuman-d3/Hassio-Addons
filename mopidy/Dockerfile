ARG BUILD_FROM
FROM $BUILD_FROM

ENV MOPIDY_VERSION='3.3.0-1'
ENV YOUTUBEDL_VERSION='2021.12.17'

RUN apk add --no-cache --repository=https://dl-cdn.alpinelinux.org/alpine/edge/community --repository=https://dl-cdn.alpinelinux.org/alpine/edge/main \
      python3-dev=3.9.5-r1 \
      py3-gst=1.18.4-r1 \
      py3-pip=20.3.4-r1
      
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
RUN apt-get update && apt-get -y --no-install-recommends install wget gnupg &&\
    mkdir -p /usr/local/share/keyrings/ &&\
    wget -q -O /usr/local/share/keyrings/mopidy-archive-keyring.gpg \
    https://apt.mopidy.com/mopidy.gpg &&\
    wget -q -O /etc/apt/sources.list.d/mopidy.list "https://apt.mopidy.com/buster.list" &&\
    apt-get update &&\
    apt-get -y --no-install-recommends install \
        mopidy=${MOPIDY_VERSION} \
        jq \
        python3-pip \
        python3-setuptools \
        python3-wheel \
        gstreamer1.0-alsa \
        gstreamer1.0-plugins-bad \
        sudo &&\
    apt-get -y autoremove && apt-get clean && rm -rf /var/lib/apt/lists/*

RUN pip3 install youtube-dl certifi pafy requests==2.28.1 charset-normalizer idna importlib-metadata pykka pytube ytmusicapi Mopidy-Iris Mopidy-Local Mopidy-SoundCloud Mopidy-InternetArchive Mopidy-YTMusic

FROM build-base AS mopidy-spotify

# hadolint ignore=DL3003,SC2016
RUN apk add --no-cache wget=1.21.1-r1 build-base=0.5-r2 libffi-dev=3.3-r2 py3-wheel=0.36.2-r0 &&\
      (([[ "$(uname -m)" == "armv7l" ]] && wget -q --show-progress https://mopidy.github.io/libspotify-archive/libspotify-12.1.51-Linux-armv7-release.tar.gz -O libspotify.tar.gz) ||\
      ([[ "$(uname -m)" == "x86_64" ]] && wget -q --show-progress https://mopidy.github.io/libspotify-archive/libspotify-12.1.51-Linux-x86_64-release.tar.gz -O libspotify.tar.gz)) &&\
      tar -xzf libspotify.tar.gz  &&\
      cd libspotify-* && sed -i 's|install -T|install|g;s|ldconfig|ldconfig $(prefix)/lib/libspotify.so\n\tldconfig $(prefix)/lib/libspotify.so.12\n\tldconfig $(prefix)/lib/libspotify.so.12.1.51|g' Makefile &&\
      make install prefix=/usr/local

# hadolint ignore=DL3059
RUN pip3 install --no-cache-dir --user \
      pyspotify==2.1.3 \
      Mopidy-Spotify==4.1.1 &&\
      ls -al /root/.local/lib/python3.9/site-packages  
      
COPY --from=mopidy-spotify /root/.local/lib/python3.9/site-packages /usr/lib/python3.9/site-packages/
COPY --from=mopidy-spotify /usr/local/lib/libspotify.so.12.1.51 /usr/local/lib/
COPY --from=mopidy-spotify /usr/local/lib/pkgconfig/libspotify.pc /usr/local/lib/pkgconfig/

# hadolint ignore=DL3003
RUN cd /usr/local/lib/ && \
      ln -sf libspotify.so.12.1.51 libspotify.so.12 && \
      ln -sf libspotify.so.12.1.51 libspotify.so

# YouTube
# hadolint ignore=DL3013
RUN pip3 install https://github.com/natumbri/mopidy-youtube/archive/develop.zip

# Let mopidy run local scans
RUN echo "mopidy ALL=NOPASSWD: /usr/local/lib/python3.9/dist-packages/mopidy_iris/system.sh" >> /etc/sudoers

COPY mopidy.conf /share/mopidy/mopidy.conf

COPY run.sh /
RUN chmod a+x /run.sh

EXPOSE 6600 6680
CMD [ "/run.sh" ]
