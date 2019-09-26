### kubernetes for the hobbyist

Kubernetes is very complex and has many parts. It is extremely modular and that is great! It causes some pain though if you want to spin it up yourself at home and just have an orchestrator for an old pc/server. I recommend using swarm for simplicity in reality, but maybe you just want to learn the basics of kubernetes. Well, lets continue into kubernetes.

### what is kubernetes

Kubernetes is an orchestration software for managing the lifecycle of containers. It's goal is to take a number of servers and abstracting them into resources for deploying containers to use as many of those resources as possible. Kubernetes is very modular and writen in golang. It provides support for various container runtimes (docker, containerd, etc), networking infrastructure, storage solutions, etc. If the existing integrations do not work for you they are all also defined as golang interfaces which you can implement yourself and drop in replacing various parts.

### How will we deploy kubernetes?

We will be doing something dumb, a single node kubernetes cluster. This cluster will also be running the infrastructure required for kubernetes to run. I do not recommend this approach at all for anything but learning. It is a much better idea to have a multiple server setup so that the control server tools can run seperate from worker nodes for both resource contention and redundency.

We will be doing this on an ubuntu 18.04 LTS server

To start we need to install the required/convenient tools to run and set up kubernetes.

First we need a CRI, container runtime interace. This can be any container software that satisfies the interface for managing the containerized applications such as containerd, docker, rt. The interface requires the ability to create, list, remove, start, and stop containers and pod level abstractions as well as exec, attach, port forward, update runtime configuration, and get the status of the runtime container software (https://github.com/kubernetes/kubernetes/blob/release-1.5/pkg/kubelet/api/v1alpha1/runtime/api.proto#L7). If you know docker you are likely familiar with all but the pod abstraction requirements. In this case we will use docker as our CRI.

```
- name: add docker apt-key
  apt_key:
    url: https://download.docker.com/linux/ubuntu/gpg
    state: present

- name: Set the stable docker repository
  apt_repository:
    repo: "deb [arch=amd64] https://download.docker.com/linux/ubuntu {{ ansible_lsb.codename }} edge"
    state: present
    update_cache: yes

# install Container Runtime Interface (docker)
- name: install CRI docker
  package:
    name: docker-ce=18.06.1~ce~3-0~ubuntu
    state: present
```

- kubelet - FILL THIS OUT
- kubeadm - This is a convenience tool for bootstraping kubernetes nodes/masters and managing cluster membership
- kubectl - The tool used for interacting with the kubernetes cluster for querying, modifications, and configuration.

```
- name: add kube utilities apt-key
  apt_key:
    url: https://packages.cloud.google.com/apt/doc/apt-key.gpg
    state: present

# bionic fails atm
- name: add kube apt repo
  apt_repository:
    repo: "deb http://apt.kubernetes.io/ kubernetes-xenial main"
    state: present
    update_cache: yes

# install kube utilitites
- name: install kubernetes utilities kubeadmn kubelet kubectl
  package:
    name: "{{ item }}"
    state: present
  with_items:
    - kubeadm
    - kubelet
    - kubectl
```

Now we have all the tools required to install kubernetes, but to install kubernetes we need to set up our server correctly. Kubernetes requires a few things

- swap to be disabled
- a static ip? maybe?
- dns

Kubernetes will not install or run if swap is enabled so we have to disable it both now and make sure it is disabled on reboot. This is not difficult to do.

```
- name: disable swap now
  shell: swapoff -a
  when: ansible_swaptotal_mb > 0

- name: disable swap for reboot
  mount:
    name: "{{ item }}"
    fstype: swap
    state: absent
  with_items:
    - swap
    - none
```

### links

(my other docker post with information about it)
(my kubernetes bakery module)
