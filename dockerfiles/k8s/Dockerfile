FROM almalinux:9.4

COPY ./kubernetes.repo /etc/yum.repos.d/

VOLUME ["/var/lib/docker"]

RUN yum install -v -y yum-utils kubectl-1.31.2 kubeadm-1.31.2 kubelet-1.31.2 net-tools \
	#&& mv -f /etc/systemd/system/kubelet.service.d/10-kubeadm.conf /etc/systemd/system/kubelet.service \
	&& yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo \
	&& yum install -y docker-ce git bash-completion \
	&& sed -i -e '4d;5d;8d' /lib/systemd/system/docker.service \
	&& yum clean all

RUN curl -Lf -o /usr/bin/jq https://github.com/stedolan/jq/releases/download/jq-1.5/jq-linux64 \
	&& curl -Lf -o /usr/bin/docker-compose https://github.com/docker/compose/releases/download/1.21.0/docker-compose-$(uname -s)-$(uname -m) \
	&& chmod +x /usr/bin/jq /usr/bin/docker-compose


VOLUME ["/var/lib/kubelet"]

COPY ./kube* /etc/systemd/system/
COPY ./entrypoint.sh /entrypoint.sh
COPY ./wrapkubeadm.sh /usr/local/bin/kubeadm
COPY ./tokens.csv /etc/pki/tokens.csv
COPY ./daemon.json /etc/docker/
COPY ./resolv.conf.override /etc/
COPY ./docker.service /usr/lib/systemd/system/
COPY ./.bashrc /root/

COPY ${MOTD:-motd} /etc/motd


RUN echo 'source <(kubectl completion bash)' >>~/.bashrc \
	&& kubectl completion bash >> /etc/bash_completion.d/kubectl

RUN mkdir -p /root/.kube && ln -s /etc/kubernetes/admin.conf /root/.kube/config \
	&& rm -f /etc/machine-id

RUN yum install -y procps-ng

WORKDIR /root
COPY ./systemctl /usr/bin/systemctl

ENTRYPOINT [ "/entrypoint.sh" ]

CMD mount --make-shared / \
    && systemctl start docker \
    && systemctl start kubelet \
    && while true; do script -q -c "/bin/bash -l" /dev/null; done