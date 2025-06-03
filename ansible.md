1. first  we need to install ansible using this command :

```
sudo apt install -y ansible sshpass
```
2. we need to set up the inventory file where all of the affected server's ip adresses will be

```
nano ~/inventory.ini
```
3. example of the content of the inventory
```
172.232.50.22 ansible_user=root

[master]
172.232.50.22 ansible_user=root

[workers]
172.232.162.75 ansible_user=root

[k8s_cluster:children]
master
workers


```
4. we list all the ip adresses :
```
ansible all --list-hosts -i ~/inventory.ini
```

5. check if the ansible server can reach into the other servers :

```
ANSIBLE_HOST_KEY_CHECKING=False ansible all -m ping -i ~/inventory.ini -k OR ansible all -m ping -i inventory.ini -k 
```

6. we create the asible playbook where we list all the configurations that we want to take place :

```
nano ~/k8s-setup.yml
```

7. example of the content of the playbook to setup a k8 cluster

```
---
- name: Set up Kubernetes cluster
  hosts: k8s_cluster
  become: yes
  tasks:
    # 1. Disable swap
    - name: Disable swap immediately
      shell: |
        swapoff -a
        sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

    # 2. Install Docker
    - name: Install Docker
      apt:
        name: docker.io
        state: present
        update_cache: yes
      notify: Start and enable Docker  # Fixed: No indentation for `notify`

    # 3. Add user to docker group
    - name: Add user to docker group
      user:
        name: root
        groups: docker
        append: yes

    # 4. Install K8s dependencies
    - name: Install dependencies
      apt:
        name:
          - apt-transport-https
          - ca-certificates
          - curl
          - gnupg
        state: present

    # 5. Add Kubernetes repo
    - name: Add Kubernetes GPG key
      shell: |
        mkdir -p /etc/apt/keyrings
        curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
        echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | tee /etc/apt/sources.list.d/kubernetes.list

    # 6. Install kubeadm, kubelet, kubectl
    - name: Install Kubernetes packages
      apt:
        name:
          - kubeadm=1.28.1-1.1
          - kubelet=1.28.1-1.1
          - kubectl=1.28.1-1.1
        state: present
        update_cache: yes
      notify: Hold Kubernetes packages  # Fixed: No indentation for `notify`

  handlers:
    - name: Start and enable Docker
      systemd:
        name: docker
        state: started
        enabled: yes

    - name: Hold Kubernetes packages
      shell: apt-mark hold kubeadm kubelet kubectl

```
8. the command to execute the playbook

```
ansible-playbook -i ~/inventory.ini ~/k8s-setup.yml -k
```


   









   
