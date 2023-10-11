FROM ubuntu:23.04 AS tool

COPY *.crt /usr/local/share/ca-certificates/

RUN echo "Update packages, install required packages and update trust store" && \
    apt-get update && \
    apt-get install --yes --no-install-recommends curl unzip ca-certificates && \
    apt-get clean && \
    update-ca-certificates

FROM tool AS terraform
ARG ARG_TERRAFORM_VERSION=1.5.7
RUN echo "Download and unpack terraform binary" && \
    curl -kSL -o /tmp/terraform.zip https://releases.hashicorp.com/terraform/${ARG_TERRAFORM_VERSION}/terraform_${ARG_TERRAFORM_VERSION}_linux_amd64.zip && \
    ls -al /tmp && \
    unzip -q /tmp/terraform.zip -d /tmp && \
    chmod a+x /tmp/terraform && \
    mv /tmp/terraform /usr/bin/terraform && \
    terraform --version

FROM tool AS packer
ARG ARG_PACKER_VERSION=1.9.4
RUN echo "Download and unpack packer binary" && \
    curl -kSL https://releases.hashicorp.com/packer/${ARG_PACKER_VERSION}/packer_${ARG_PACKER_VERSION}_linux_amd64.zip -o /tmp/packer.zip && \
    unzip -q /tmp/packer.zip -d /tmp && \
    chmod a+x /tmp/packer && \
    mv /tmp/packer /usr/bin/packer && \
    packer --version

FROM tool AS terragrunt
ARG ARG_TERRAGRUNT_VERSION=v0.52.1
RUN echo "Download terragrunt binary" && \
    curl -kSL https://github.com/gruntwork-io/terragrunt/releases/download/${ARG_TERRAGRUNT_VERSION}/terragrunt_linux_amd64 -o /tmp/terragrunt_linux_amd64  && \
    chmod a+x /tmp/terragrunt_linux_amd64  && \
    mv /tmp/terragrunt_linux_amd64 /usr/bin/terragrunt  && \
    terragrunt --version

FROM tool AS kubectl
ARG ARG_KUBECTL_VERSION=1.28.2
RUN echo "Download kubectl binary" && \
    curl -fsSL -o /usr/local/bin/kubectl https://storage.googleapis.com/kubernetes-release/release/v$ARG_KUBECTL_VERSION/bin/linux/amd64/kubectl && \
    chmod a+x /usr/local/bin/kubectl

FROM tool AS helm
ARG ARG_HELM_VERSION=3.13.0
RUN echo "Download and unpack helm binaries" && \
    curl -fsSL -o /tmp/helm-linux-amd64.tar.gz https://get.helm.sh/helm-v$ARG_HELM_VERSION-linux-amd64.tar.gz && \
    mkdir -p /tmp/helm && \
    tar -xzf /tmp/helm-linux-amd64.tar.gz -C /tmp/helm && \
    mv /tmp/helm/linux-amd64/helm /usr/local/bin/helm && \
    chmod a+x /usr/local/bin/helm

FROM tool AS awscli
RUN echo "Download and unpack AWS CLI binaries" && \
    curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o /tmp/awscliv2.zip && \
    unzip -q /tmp/awscliv2.zip -d /tmp && \
    /tmp/aws/install

FROM ubuntu:23.04

ARG ARG_KUBECTL_VERSION=1.28.2
ARG ARG_HELM_VERSION=3.13.0
ARG ARG_KUBECTL_HOME=/root/.kube
ARG ARG_HELM_HOME=/root/.helm

LABEL maintainer="michael.theis@msg.group"
LABEL ${project.groupId}.${project.artifactId}.project="CloudTrain"
LABEL ${project.groupId}.${project.artifactId}.version="${project.version}"
LABEL ${project.groupId}.${project.artifactId}.description="${project.description}"
LABEL ${project.groupId}.${project.artifactId}.kubectl.version="$ARG_KUBECTL_VERSION"
LABEL ${project.groupId}.${project.artifactId}.helm.version="$ARG_HELM_VERSION"

ENV \
# Set default encoding to UTF-8
  LANG=C.UTF-8

# copy Terraform binary from build terraform
COPY --from=terraform /usr/bin/terraform /usr/bin/terraform
# copy Packer binary from build packer
COPY --from=packer /usr/bin/packer /usr/bin/packer
# copy Terragrunt binary from build terragrunt
COPY --from=terragrunt /usr/bin/terragrunt /usr/bin/terragrunt
# copy kubectl binaries from build kubectl
COPY --from=kubectl /usr/local/bin/kubectl /usr/local/bin/kubectl
# copy helm binaries from build helm
COPY --from=helm /usr/local/bin/helm /usr/local/bin/helm
# copy AWS CLI binaries from build awscli
COPY --from=awscli /tmp/aws/dist /opt/aws-cli

RUN echo "Update system packages" && \
    apt-get update && \
    echo "Install missing tools" && \
    apt-get install --yes groff && \
    echo "Clean up after installation" && \
    apt-get clean && \
    apt-get autoremove --purge && \
    rm -rf /var/lib/apt/lists/* && \
    echo "Install Terraform" && \
    terraform --version && \
    echo "Install Packer" && \
    packer --version && \
    echo "Install Terragrunt" && \
    terragrunt --version && \
    echo "Install kubectl" && \
    mkdir -p $ARG_KUBECTL_HOME && \
    echo "Install helm" && \
    mkdir -p $ARG_HELM_HOME && \
    helm version && \
    echo "Install AWS CLI v2" && \
    ln -sf /opt/aws-cli/aws /usr/local/bin/aws && \
    ln -sf /opt/aws-cli/aws_completer /usr/local/bin/aws_completer

COPY docker-entrypoint.sh /usr/bin/

RUN echo "Change access rights on copied shell scripts" && \
    chmod u+x /usr/bin/docker-entrypoint.sh

# Unfortunately we have to use hardcoded paths here if we want to pass arguments to the script
ENTRYPOINT ["/usr/bin/docker-entrypoint.sh"]
