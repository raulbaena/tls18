FROM fedora:27
LABEL author="@edt ASIX M11-SAD"
LABEL description="Sevrer https 2018-2019"
RUN dnf -y install httpd nmap iputils iproute tree mod_ssl vim
RUN mkdir /opt/docker
COPY * /opt/docker/
RUN chmod +x /opt/docker/startup.sh /opt/docker/install.sh
WORKDIR /opt/docker
CMD [ "/opt/docker/startup.sh" ]
