# Access remote Minikube

I have a PC with enough Memory/CPU where I can run a Minikube K8s cluster of a size that wouldn't fit on my laptop. I do however, like to be able to access that local K8s cluster from my laptop, and since what I deploy there uses just a local docker registry, I'd also like to be able to use the Docker daemon on the PC when building images from my Laptop.

For Docker, this is quite straightforward and once I had SSH access to the PC from the laptop, I only needed to define the environment variable `DOCKER_HOST` as `ssh://username@pc-address`. Note that this means when you issue a Docker build command, the build context is sent to the daemon which is not a local process and this could potentially take much longer.

For accessing the Minikube cluster, a simple SSH port-forward worked along with a modified copy of the original kubeconfig file.

I'd need the internal IP of the VM where Minikube is running:
```shell
$ kubectl get nodes -owide
NAME       STATUS   ROLES           AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
minikube   Ready    control-plane   33d   v1.25.2   192.168.49.2   <none>        Ubuntu 20.04.5 LTS   5.15.0-48-generic   docker://20.10.18
```

And port-forward with:
```ssh
ssh -NfL 8443:192.168.49.2:8443 username@pc-address 
```

I then copied over the kubeconfig file to my laptop and changed one line in it:
```
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /home/XXXXX/.minikube/ca.crt
    extensions:
    - extension:
        last-update: Sat, 03 Dec 2022 12:02:59 CET
        provider: minikube.sigs.k8s.io
        version: v1.27.1
      name: cluster_info
    server: https://127.0.0.1:8443 #--> Localhost instead of the Minikube VM IP
  name: minikube
contexts:
- context:
    cluster: minikube
    extensions:
    - extension:
        last-update: Sat, 03 Dec 2022 12:02:59 CET
        provider: minikube.sigs.k8s.io
        version: v1.27.1
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
    client-certificate: /home/XXXXX/.minikube/profiles/minikube/client.crt
    client-key: /home/XXXXX/.minikube/profiles/minikube/client.key

```

I also needed to copy over the root certificate and client secret and certificate, which are provided as file addresses in the kubeconfig file. Alternatively, I think one could just replace the content of these three entries with the corresponding file contents, but I think then the key names are different (namely, `certificate-authority-data`, `client-certificate-data` and `client-key-data`).

-- 04.12.2022