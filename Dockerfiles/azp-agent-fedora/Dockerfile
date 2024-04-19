FROM quay.io/podman/stable as base
ENV TARGETARCH="linux-x64"

# Set up folders
RUN mkdir -p /opt /data

# Repo credential
## Temurin
COPY adoptium.repo /etc/yum.repos.d/adoptium.repo
## Powershell
RUN rpm --import https://packages.microsoft.com/keys/microsoft.asc
RUN rpm --import https://packages.microsoft.com/keys/microsoft.asc
RUN curl https://packages.microsoft.com/config/rhel/7/prod.repo | tee /etc/yum.repos.d/microsoft.repo
RUN dnf makecache

# Install software system
## By package manager
RUN dnf update -y && dnf upgrade -y
RUN dnf install -y \
    libicu \
    which \
    telnet \
    wget \
    git \
    jq \
    tree \
    nano \
    vim \
    zip \
    temurin-17-jdk \
    temurin-11-jdk \
    nodejs \
    npm \
    dotnet-sdk-6.0 \
    powershell
## By manual
RUN mkdir -p /opt/maven && \
    wget -qO- https://repo.maven.apache.org/maven2/org/apache/maven/apache-maven/3.6.3/apache-maven-3.6.3-bin.tar.gz | tar xvz -C /opt/maven

# Expose environment variables
## Override PATH var
ENV MAVEN_HOME=/opt/maven/apache-maven-3.6.3
ENV JAVA_HOME=/usr/lib/jvm/temurin-17-jdk
ENV PATH="${PATH}:${MAVEN_HOME}/bin:${JAVA_HOME}/bin"
## Change the location contains libraries of platforms
ENV MAVEN_OPTS="-Dmaven.repo.local=/data/.m2"
ENV CONDA_ENVS_DIRS=/data/.conda/envs
ENV NUGET_PACKAGES=/data/.nuget
RUN npm config set prefix /data/.npm

FROM base as agent
WORKDIR /azp/

RUN chown -R podman:podman /opt /data /azp /home/podman

COPY ./start.sh ./
RUN chmod +x ./start.sh

USER podman
# Install software user
## Miniconda
RUN mkdir -p /opt/miniconda3 \
    && wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O /opt/miniconda3/miniconda.sh
RUN bash /opt/miniconda3/miniconda.sh -b -u -p /opt/miniconda3
RUN rm -rf /opt/miniconda3/miniconda.sh
RUN /opt/miniconda3/bin/conda init && /opt/miniconda3/bin/conda config --set auto_activate_base false

ENTRYPOINT ./start.sh