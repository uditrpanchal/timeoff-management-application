FROM amazon/aws-cli:2.9.7

USER root

RUN yum -y install tar gzip && \
    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl" && \
    chmod +x ./kubectl && \
    mv ./kubectl /usr/local/bin/kubectl 

RUN touch /.aws/credentials && \
    chmod +x /.aws/credentials

ENTRYPOINT [""]