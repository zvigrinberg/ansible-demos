FROM ubuntu:16.04

USER root

RUN apt-get update && apt-get install -y openssh-server python3 iputils-ping iproute2 netcat \
    &&  mkdir /var/run/sshd \
    &&  echo 'root:Redhat1234#' | chpasswd

COPY $HOME/.ssh/id_ed25519.pub /tmp/id_ed25519.pub
RUN mkdir -p ~/.ssh \ 
    && cat /tmp/id_ed25519.pub > ~/.ssh/authorized_keys

EXPOSE 22
CMD ["/usr/sbin/sshd", "-D"]
