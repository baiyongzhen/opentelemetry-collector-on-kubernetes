# opentelemetry-collector-on-kubernetes

# Start Vagrant
```bash
vagrant up 
vagrant up minikube
vagrant ssh minikube
```

# Start Minikube
```bash
minikube start --driver=docker --listen-address='0.0.0.0' --apiserver-ips='192.168.56.161'

# 가상머신 CPU 2개 이상 설정이 필요함.
vagrant@minikube:~$ minikube start --driver=docker --listen-address='0.0.0.0' --apiserver-ips='192.168.56.161'
😄  minikube v1.31.2 on Ubuntu 18.04 (vbox/amd64)
✨  Using the docker driver based on user configuration
📌  Using Docker driver with root privileges
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
🔥  Creating docker container (CPUs=2, Memory=2200MB) ...
💡  minikube is not meant for production use. You are opening non-local traffic
❗  Listening to 0.0.0.0. This is not recommended and can cause a security vulnerability. Use at your own risk
...

```

#  Lens 에 등록할 kubeconfig 작성함.
```yaml
# minikube start 이후에 생성된 key 값을 복사합니다.
# minikube port
vagrant@minikube:~$ docker ps -a
8a023e41b3ef   gcr.io/k8s-minikube/kicbase:v0.0.40   "/usr/local/bin/entr…"   7 hours ago      Up 18 minutes   0.0.0.0:32772->22/tcp, 0.0.0.0:32771->2376/tcp, 0.0.0.0:32770->5000/tcp, 0.0.0.0:32769->8443/tcp, 0.0.0.0:32768->32443/tcp   minikube

# minikube kubeconfig 등록하기
apiVersion: v1
clusters:
- cluster:
    certificate-authority: <your-host-pc-dir>\ca.crt
    extensions:
    - extension:
        last-update: Mon, 28 Aug 2023 09:31:37 UTC
        provider: minikube.sigs.k8s.io
        version: v1.31.2
      name: cluster_info
    server: https://192.168.56.161:32769
  name: minikube
contexts:
- context:
    cluster: minikube
    extensions:
    - extension:
        last-update: Mon, 28 Aug 2023 09:31:37 UTC
        provider: minikube.sigs.k8s.io
        version: v1.31.2
      name: context_info
    namespace: default
    user: minikube
  name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate: <your-host-pc-dir>\client.crt
    client-key: <your-host-pc-dir>\client.key

```


# Install OpenTelemetry operator
```bash
To install the operator in an existing cluster, make sure you have cert-manager installed and run:
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.12.0/cert-manager.yaml

# Getting started
kubectl apply -f https://github.com/open-telemetry/opentelemetry-operator/releases/latest/download/opentelemetry-operator.yaml

```



# 예비 VM 환경 설정
```bash
  config.vm.define :backend do |host|
    host.vm.box = "bento/ubuntu-18.04"
    host.vm.hostname = "backend"
    host.vm.network :private_network, ip: "192.168.56.162"
    host.vm.provision :shell, path: "scripts/debian_bootstrap.sh"
    # boot timeout
    host.vm.boot_timeout = 300

    # Set system settings
    host.vm.provider :virtualbox do |vb|
        vb.customize ["modifyvm", :id, "--memory", "2048"]
        vb.customize ["modifyvm", :id, "--cpus", "1"]
        vb.customize ['modifyvm', :id, '--cableconnected1', 'on']
    end
  end

```