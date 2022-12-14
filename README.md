<h1>Client Machine -</h1> 
POST Request to HeadNode: model name & image files to send request for inference using xception, one batch of images of size 64 (from a folder consisting of only images): <br>

```bash
    for J in {1}; do (curl http://128.105.145.75/inference/xception $(for I in $(ls | tail -n $((64))); do echo -F $I=@$I; done) &); done
```
to send multiple batches (say 4) of size 64:
</br>
```bash
    for J in {1..4}; do (curl http://128.105.145.75/inference/xception $(for I in $(ls | tail -n $((128))); do echo -F $I=@$I; done) &); sleep 0.3; done
```
where, `http://128.105.145.75` is address of the head node that will perform load balancing by distributing the incoming requests to available GPUs nodes
`/inference/xception` - path for xception model (at every GPU endpoint)
<br>
<br>
<h1> Load Balancer </h1>

For load balancing Nginx is used with `least_conn` strategy. <br>
Configuration Requirements:<br>
OS: Ubuntu 20.0.4 <br>
Installation steps:<br>
Install Nginx using apt: `sudo apt install nginx` <br>
If firewall is active, to allow Nginx to accept connection runs: `sudo ufw allow 'Nginx HTTP'` <br>
Ngnix starts automatically when installed with apt, to check the status of Nginx run: `systemctl status nginx`.<br>
If the Nginx is running that means you have successfully install it.
<br>
The Nginx is located in `/etc/` folder. We have to update the `nginx.conf` file located in `/etc/nginx/` folder. To do so replace the `nginx.conf` file with the one provided in the repository.
After replacing the file, update the IP addresses of the worker servers in the `nginx.conf` file.  The ip address of the server are located under the `upstream myapp1` inside `http` section of the file. <br>
Now kill the Nginx instance running by using: ` sudo killall nginx`
<br>
Restart the Nginx server with the location of your conf file: ` sudo /usr/sbin/nginx -c /etc/nginx/nginx.conf `
<br>
Check the status of Nginx using: `systemctl status nginx` <br>
If you get a similar result, that means you nginx server is up and running:
```
nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
     Active: inactive (dead) since Fri 2022-12-02 12:49:50 CST; 5 days ago
       Docs: man:nginx(8)
    Process: 530548 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
    Process: 530556 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
    Process: 530634 ExecStop=/sbin/start-stop-daemon --quiet --stop --retry QUIT/5 --pidfile /run/nginx.pid (code=exited, status=0/SUCCESS)
   Main PID: 530557 (code=exited, status=0/SUCCESS)
```
<br>

<h1> GPU Nodes </h1>
If nvidia and docker dependencies are not yet installed install them.

```bash
#Install CUDA
sudo apt-get install linux-headers-$(uname -r)
distribution=$(. /etc/os-release;echo $ID$VERSION_ID | sed -e 's/\.//g')
wget https://developer.download.nvidia.com/compute/cuda/repos/$distribution/x86_64/cuda-keyring_1.0-1_all.deb
sudo dpkg -i cuda-keyring_1.0-1_all.deb

sudo apt-get update
sudo apt-get -y install cuda-drivers

#REBOOT & test with nvidia-smi

#install docker if not installed already
curl -fsSL https://get.docker.com | sudo sh

#install Nvidia docker
#The following 5 lines are a single command
distribution=$(. /etc/os-release;echo $ID$VERSION_ID) \
      && curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
      && curl -s -L https://nvidia.github.io/libnvidia-container/$distribution/libnvidia-container.list | \
            sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
            sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
            
       
sudo apt-get update
sudo apt-get install -y nvidia-docker2
sudo systemctl restart docker

#test with:
sudo docker run --rm --gpus all nvidia/cuda:11.0.3-base-ubuntu20.04 nvidia-smi
```





GPU Nodes - execute inference task with image files
    
Steps
Download docker image

```bash
sudo docker pull mnj98/gpu-serverless:latest
```
 Run docker image
 Use the run.sh script 
