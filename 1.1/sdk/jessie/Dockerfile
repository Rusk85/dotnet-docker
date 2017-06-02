FROM buildpack-deps:jessie-scm

# Install .NET CLI dependencies
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        libc6 \
        libcurl3 \
        libgcc1 \
        libgssapi-krb5-2 \
        libicu52 \
        liblttng-ust0 \
        libssl1.0.0 \
        libstdc++6 \
        libunwind8 \
        libuuid1 \
        zlib1g \
	openssh-server \
	openjdk-7-jdk \
	sudo \
    && rm -rf /var/lib/apt/lists/*

# Install .NET Core SDK
ENV DOTNET_SDK_VERSION 1.0.4
ENV DOTNET_SDK_DOWNLOAD_URL https://dotnetcli.blob.core.windows.net/dotnet/Sdk/$DOTNET_SDK_VERSION/dotnet-dev-debian-x64.$DOTNET_SDK_VERSION.tar.gz

RUN curl -SL $DOTNET_SDK_DOWNLOAD_URL --output dotnet.tar.gz \
	&& mkdir -p /usr/share/dotnet \
	&& tar -zxf dotnet.tar.gz -C /usr/share/dotnet \
	&& rm dotnet.tar.gz \
	&& ln -s /usr/share/dotnet/dotnet /usr/bin/dotnet \
	&& sed -i 's|session    required     pam_loginuid.so|session    optional     pam_loginuid.so|g' /etc/pam.d/sshd \
	&& sed -i "23s/.*/%sudo ALL=NOPASSWD: ALL/" /etc/sudoers \
	&& mkdir -p /var/run/sshd \
	&& ssh-keygen -A 
	

ENV USER=jenkins
ENV J_GRP=jenkins
ENV J_HOME=/home/jenkins/
ENV DIR_SSH=${J_HOME}/.ssh/
ENV AUTH_KEYS=${J_HOME}/.ssh/authorized_keys

COPY files/authorized_keys $AUTH_KEYS

RUN adduser --quiet $USER \
	&& usermod -aG sudo $USER \
	&& echo "jenkins:jenkins" | chpasswd \
	&& mkdir -p $DIR_SSH \
	&& chown -R ${USER}:${J_GRP} $J_HOME \
	&& chmod 600 $AUTH_KEYS \
	&& chmod 700 $DIR_SSH

# Trigger the population of the local package cache
ENV NUGET_XMLDOC_MODE skip
RUN mkdir warmup \
    && cd warmup \
    && dotnet new \
    && cd .. \
    && rm -rf warmup \
    && rm -rf /tmp/NuGetScratchi

EXPOSE 22

CMD ["/usr/sbin/sshd", "-D"]
