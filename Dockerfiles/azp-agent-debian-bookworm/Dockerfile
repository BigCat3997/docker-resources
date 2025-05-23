FROM debian:bookworm

ENV TARGETARCH="linux-x64"
ARG ACTIVE_USER="agent"

# Install software system
## By package manager
RUN apt update -y && apt upgrade -y
RUN apt install -y \
        apt-transport-https \
        apt-utils \
        gnupg \
        ca-certificates \
        lsb-release \
        software-properties-common \
        iputils-ping \
        telnet \
        wget \
        curl \
        git \
        jq \
        libxml2-utils \
        tree \
        nano \
        vim \
        zip \
        sudo

# Repo credential
## Temurin JDK
RUN wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | apt-key add -
RUN echo "deb https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | tee /etc/apt/sources.list.d/adoptium.list
## Docker
RUN install -m 0755 -d /etc/apt/keyrings
RUN curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
RUN chmod a+r /etc/apt/keyrings/docker.asc
RUN echo \
        "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
        $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
        tee /etc/apt/sources.list.d/docker.list > /dev/null
## MS software
RUN wget https://packages.microsoft.com/config/debian/12/packages-microsoft-prod.deb -O packages-microsoft-prod.deb && \
        dpkg -i packages-microsoft-prod.deb && \
        rm -rf packages-microsoft-prod.deb
## Npm
RUN curl -fsSL https://deb.nodesource.com/setup_18.x | bash -

# Update credential for repo before install software here
RUN apt-get update

RUN apt-get install -y \
        docker-ce \
        docker-ce-cli \
        containerd.io \
        docker-buildx-plugin \
        docker-compose-plugin \
        maven \
        temurin-21-jdk \
        temurin-17-jdk \
        temurin-11-jdk \
        nodejs

# Install .NET 6 SDK and Runtime
RUN apt-get install -y dotnet-sdk-6.0 dotnet-runtime-6.0

# Install .NET 8 SDK and Runtime
RUN wget https://dot.net/v1/dotnet-install.sh -O dotnet-install.sh && \
        chmod +x dotnet-install.sh && \
        ./dotnet-install.sh --channel 8.0 --install-dir /usr/share/dotnet8 && \
        ln -s /usr/share/dotnet8/dotnet /usr/bin/dotnet8 && \
        rm dotnet-install.sh

## Az
RUN curl -sL https://aka.ms/InstallAzureCLIDeb | bash

# Azcopy
RUN apt-get install -y azcopy

## Kubectl
RUN curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl" && \
        install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl && \
        rm -rf kubectl

## Helm
RUN curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 > get_helm.sh && \
        chmod 700 get_helm.sh && \
        ./get_helm.sh && \
        rm -rf get_helm.sh

# Terraform
RUN wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg > /dev/null
RUN echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
        https://apt.releases.hashicorp.com $(lsb_release -cs) main" | \
        tee /etc/apt/sources.list.d/hashicorp.list
RUN apt-get update && apt-get install -y terraform

WORKDIR /azp/

COPY ./start.sh ./
RUN chmod +x ./start.sh

RUN mkdir -p /opt /data
RUN useradd -m -s /bin/bash $ACTIVE_USER
RUN usermod -aG root,docker,sudo agent
RUN chown -R $ACTIVE_USER:$ACTIVE_USER /azp /opt /data
RUN echo "$ACTIVE_USER ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

USER $ACTIVE_USER
# Install software user
## Miniconda
RUN mkdir -p /opt/miniconda3 && \
        wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O /opt/miniconda3/miniconda.sh
RUN bash /opt/miniconda3/miniconda.sh -b -u -p /opt/miniconda3
RUN rm -rf /opt/miniconda3/miniconda.sh
RUN /opt/miniconda3/bin/conda init && /opt/miniconda3/bin/conda config --set auto_activate_base false
## Azure agent
RUN curl -LsS https://vstsagentpackage.azureedge.net/agent/3.238.0/vsts-agent-linux-x64-3.238.0.tar.gz | tar -xz & wait $!

# Expose environment variables
## Change the location contains libraries of platforms
ENV AGENT_ALLOW_RUNASROOT=1
ENV CONDA_ENVS_DIRS=/data/.conda/envs
ENV NUGET_PACKAGES=/data/.nuget
ENV JAVA_JDK21_HOME=/usr/lib/jvm/temurin-21-jdk-amd64
ENV JAVA_JDK17_HOME=/usr/lib/jvm/temurin-17-jdk-amd64
ENV JAVA_JDK11_HOME=/usr/lib/jvm/temurin-11-jdk-amd64
ENV MAVEN_OPTS="-Dmaven.repo.local=/data/.m2"
ENV DOTNET_ROOT=/usr/share/dotnet
ENV PATH=$PATH:/usr/share/dotnet:/usr/share/dotnet8

RUN npm config set prefix /data/.npm

ENTRYPOINT ["./start.sh"]