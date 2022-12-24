FROM ubuntu:16.04

USER root
RUN apt-get update && apt-get install -y openssh-server python3 \
    &&  mkdir /var/run/sshd \
    &&  echo 'root:mypassword' | chpasswd \
    &&  sed -i 's/PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config \
    &&  sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd
EXPOSE 22
CMD ["/usr/sbin/sshd", "-D"]
