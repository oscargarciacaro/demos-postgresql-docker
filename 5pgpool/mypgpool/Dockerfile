FROM centos:latest

RUN yum -y update && yum -y install http://www.pgpool.net/yum/rpms/3.6/redhat/rhel-7-x86_64/pgpool-II-release-3.6-1.noarch.rpm
RUN yum -y install pgpool-II-pg96

COPY pgpool.conf /etc/pgpool-II/
COPY .pcppass /root/
COPY pcp.conf /etc/pgpool-II/

#CMD ["/bin/bash"]
CMD ["/usr/bin/pgpool","-f","/etc/pgpool-II/pgpool.conf","-n"]
