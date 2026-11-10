
FROM docker:cli AS docker-cli

FROM jenkins/jenkins:lts-jdk21

USER root

# Keep Docker CLI available inside Jenkins
COPY --from=docker-cli /usr/local/bin/docker /usr/local/bin/docker

# Install prerequisites, AWS CLI v2, and kubectl
RUN apt-get update \
    && apt-get install -y --no-install-recommends curl unzip ca-certificates \
    && curl -fsSL "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o /tmp/awscliv2.zip \
    && unzip -q /tmp/awscliv2.zip -d /tmp \
    && /tmp/aws/install \
    && curl -fsSL "https://dl.k8s.io/release/v1.34.0/bin/linux/amd64/kubectl" -o /usr/local/bin/kubectl \
    && chmod +x /usr/local/bin/kubectl \
    && rm -rf /tmp/aws /tmp/awscliv2.zip \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

USER jenkins
