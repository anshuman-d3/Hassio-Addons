ARG BUILD_FROM
FROM $BUILD_FROM

ENV MOPIDY_VERSION='3.3.0-1'
ENV YOUTUBEDL_VERSION='2021.12.17'

SHELL ["/bin/bash", "-o", "pipefail", "-c"]
RUN apt-get update && apt-get -y --no-install-recommends install wget gnupg &&\
    mkdir -p /usr/local/share/keyrings/ &&\
    wget -q -O /usr/local/share/keyrings/mopidy-archive-keyring.gpg \
    https://apt.mopidy.com/mopidy.gpg &&\
    wget -q -O /etc/apt/sources.list.d/mopidy.list "https://apt.mopidy.com/buster.list" &&\
    apt-get update &&\
    DEBIAN_FRONTEND=noninteractive apt-get -y --no-install-recommends install \
        url \
        git \
        build-essential \
        libasound2-dev \
        libvorbisidec-dev \
        libvorbis-dev \
        libflac-dev \
        alsa-utils \
        libavahi-client-dev \
        avahi-daemon \
        build-essential \
        debhelper \
        dh-make \
        dh-systemd \
        quilt \
        fakeroot \
        lintian \
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
