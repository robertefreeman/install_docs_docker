# Docker EE 17.03.2ee5 Install Instructions

## Install on AWS: 1 UCP, 1 DTR, 2 Worker nodes

### Setup AWS environment
- Launch 1 * ec2 instance of RHEL 7.3
- Open the following ports to ucp server
  - 443,  *UCP web GUI*
  - 2376, *For legacy Docker swarm manager*
  - 2377, *Comms between swarm nodes*
  - 4789, *Overlay networking*
  - 7946, *Gossip based clustering*
  - 12376 - 12387, *Comms for storage, authentication, metrics*

### Setup Base Docker Server
- Copy Docker EE URL to * /etc/yum/vars/dockerurl *
```
sudo sh -c 'echo "<DOCKER-EE-URL>" > /etc/yum/vars/dockerurl'
```
- Store your RHEL version string in * /etc/yum/vars/dockerosversion *
```
sudo sh -c 'echo "<DOCKER-EE-URL>/rhel" > /etc/yum/vars/dockerurl'
```
- Install require YUM Packages
```
sudo yum update
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```
- Add required 'extras' repos --- ignore this step
```
sudo yum-config-manager --enable rhel-7-server-extras-rpms
sudo yum-config-manager --enable rhui-REGION-rhel-server-extras
```
- Add Docker EE repo
```
sudo yum-config-manager \
    --add-repo \
    <DOCKER-EE-URL>/centos/docker-ee.repo
```
- Update yum repos
```
sudo yum makecache fast
```
- Install Docker-EE
```
sudo yum -y install docker-ee
```
- Start Docker
```
sudo systemctl start docker
```

### Optional configurations - post install

- Configure docker to start on boot
```
sudo systemctl enable docker
```

- Configure to avoid use of sudo
```
sudo groupadd docker
sudo usermod -aG docker $USER
```

### Create base AMI

- Stop AWS ec2 instance
- Create AMI from instance for use creating all DTR and worker nodes
- Launch 3 ec2 instances from newly created AMI

### Install UCP on main instance
- Run ucp install command using nodes public IP
```
sudo docker run --rm -it --name ucp \
  -v /var/run/docker.sock:/var/run/docker.sock \
  docker/ucp install \
  --host-address 13.56.147.210 \
  --interactive
```
- Complete install prompts:
  - admin username: username of initial admin account
  - admin password: password of initial admin account
  - additional aliases: add any additional SANs required (including DNS/IP of LB)

### Add additional worker nodes
- Connect to target node via SSH
- Run swarm join command provide via UCP GUI
```
docker swarm join \
  --token SWMTKN-1-1f4q1trx471hi3ugey1ytv7fdjsmbphfi7vmd4mow86b5ut81q-5cgwa4sivxzat17ewkq56clvx \
  13.56.147.210:2377
```
- repeat on all target nodes

### Install DTR on worker node
- Connect to UCP Manager node via SSH
- run DTR install command
```
docker run -it --rm docker/dtr install --ucp-node ip-172-31-27-73 --ucp-insecure-tls
```
- Complete install prompts:
  - dtr-external-url: URL of the DTR host or load balancer clients use to reach DTR
  - ucp-url: The UCP URL including domain and port
  - ucp-username: username of initial admin account
  - ucp-password: password of initial admin account
