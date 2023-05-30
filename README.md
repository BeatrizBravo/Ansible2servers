# Introduction
This readme was written with the help of 
[Markdown-Cheatsheet](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet).&nbsp;<br />

Before running this Ansible project, we will set up a minimum of 2 servers.&nbsp;
A server can be our own computer running Ubuntu. For our local Windows machine, we will install [WSL](https://learn.microsoft.com/en-us/windows/wsl/install) to use Ubuntu on it, and then Asible [Asible](https://www.ansible.com/)  will be installed.

# Install WSL
Open powershell and type 
```bash
Wsl --install -d ubuntu;
```
This one line command is to install the Ubuntu distribution of Linux on Windows Subsystem for Linux (WSL). This command will automatically download and install the latest available version of Ubuntu.&nbsp;
the -d flag specifies the name of the distribution that you want to install&nbsp;

After running this command, you will need to configure your username and password for the Ubuntu installation and then start using it by running the wsl command, which will open up a command prompt with Ubuntu Linux running within Windows. You can then use Ubuntu commands to install packages, run Linux software, and more within the Windows environment.&nbsp;

To check if you have installed correcty wsl and ubuntu type the next command in powershell:
```bash
 WSL 
```
You will notice the user and the host have change in the console.
Also, there you can check if your Ansible was installed correctly.
```bash
Ansible --version
```

# Connect the servers
To connect the master server (our local machine) with the remote server o node server, we need to use ssh command.
SH (Secure Shell) command is a network protocol that provides secure encrypted communication between two hosts over an insecure network. To  connect a remote serever you can type in the command line:


```bash
ssh user@hostname 
```
&nbsp;hostname can be IP address or example.com
&nbsp;

If It is your first time with ssh, you need to create an SSH key.
1. Open powershell of your local computer and type
```bash
ssh-keygen 
```
Print your public key and copy:

```bash
cat id_rsa.pub | ssh user@remote-server 'cat >> .ssh/authorized_keys'
```
2. Open a new terminal and log in the remote -node- server. The node server is going to be configurate using Ansible. Log with ssh user@hostname.&nbsp;
Paste the ssh key of the local machine(master) to the remote server inside of the authorized_keys file.

```bash
 nano ~/.ssh/authorized_keys 
```
3. Save it.
4. Open a new terminal and type he command:
```bash
 WSL 
```
5. Check your conection with the remote server:
```bash
ssh user@hostname 
```

# Inventory

```bash
[all:vars]
ansible_user=user-name
[public]
boost-public    ansible_host=18.170.30.71


[private]
boost-private   ansible_host=10.10.10.138

[private:vars]
ansible_ssh_common_args='-J user-name@18.170.30.71'
```
Ansible inventory file, defines a list of hosts that Ansible can manage. Here's a breakdown of what each section means:

[all:vars]: defines variables that apply to all hosts in the inventory. In this case, it sets the ansible_user variable to "user-name", which specifies the username that Ansible should use when connecting to each host.

[public]: defines a group of hosts that are publicly accessible. The only host specified in this group is "boost-public", with the IP address "18.170.30.71". The ansible_host variable defines the IP address of the host.

[private]:  defines a group of hosts that are only accessible from within a private network. The only host specified in this group is "boost-private", with the IP address "10.10.10.138".

[private:vars]: defines variables that apply only to hosts in the "private" group. In this case, it sets the ansible_ssh_common_args variable, which specifies additional SSH command line arguments that Ansible should use when connecting to each host. In this example, it's using the -J option to specify that a jump host should be used to reach the hosts in the private network. Specifically, it's using the "user-name" user to jump through the "18.170.30.71" host to reach the "10.10.10.138" host.
# Playbook
## Nginx.yml
Front-end webser, public server with a reverse proxy configuration to relay HTTP requests on to the “backend” web servers.

&nbsp;
* **hosts** parameter specifies the group of hosts on which this playbook will be executed. In this case, the playbook will run on all hosts in the boost-private group.
* **become** parameter specifies that the tasks in this playbook will be executed using sudo.
* **tasks** section defines a list of actions that will be executed on the hosts. There are six tasks in this section:
	1. The first task installs the latest version of the Apache web server using the apt package manager.
	2. The second task creates a directory /var/www/html to serve as the document root for the web server, and sets the owner and group of the directory to www-data.
	3. The third task copies the index.html file from the local machine to the document root using the template module, which allows template variables to be rendered.
	4. The fourth task sets up a virtual host for Apache using the template module, which copies the configuration file for the virtual host from the local machine to the server. The template file, apache.conf, is shown at the end of your question.
	5. The fifth task enables the new virtual host using the a2ensite command.
	6. The sixth task opens port 80 in the firewall using the ufw module to allow incoming HTTP traffic.
* **handlers** section defines a list of actions that can be triggered by events in the playbook. In this case, there is one handler defined:
The restart-apache handler restarts the Apache service to apply the changes that were made to the virtual host configuration.
### apache.conf
It file defines how the Apache server will respond to requests for the site. It contains the virtual host configuration for Apache that runs on port 80, listens for incoming requests, and serves the content from /var/www/html.
## Apache.yml
Backend webserver, private server.
&nbsp;


* **hosts** parameter specifies the group of hosts on which this playbook will be executed. In this case, the playbook will run on all hosts in the boost-public group.
* **become** parameter specifies that the tasks in this playbook will be executed using sudo (i.e. as a privileged user).
* **tasks** section defines a list of actions that will be executed on the hosts. There are five tasks in this section:
	1. The first task updates the APT cache using the apt module.
	2. The second task installs Nginx using the apt module.
	3. The third task creates a directory /var/www/html to serve as the document root for the web server, and sets the owner and group of the directory to user-name.
	4. The fourth task deletes the default Nginx site configuration file from /etc/nginx/sites-enabled.
	5. The fifth task uses the template module to copy a custom site.conf.j2 configuration file for Nginx to /etc/nginx/sites-enabled.
* **handlers** section defines a list of actions that can be triggered by events in the playbook. In this case, there is one handler defined, which is commented out:
	The restart nginx handler restarts the Nginx service to apply the changes that were made to the Nginx site configuration.
### site.conf.j2 
It file contains the configuration for the Nginx virtual site. It specifies that Nginx should listen on port 80 and serve files from the /var/www/html directory. Additionally, it sets up the location directive that will serve the root index.html file.
### sync.yml
It contains one task that copies the index.html file to the /var/www/html directory on the remote server using the synchronize module. This task does not require root privileges as mentioned in become: no.


# Running Ansible

After having the inventory and playbook ready, we can automate the configuration of our servers.&nbsp;

## Front-end configuration
1. Update your user-name and hostname in all related files.
2. Open the terminal with the corresponding path of the repository that we are going to execute.
3. Run ubuntu with the command wsl
4. Execute Ansible: 
```bash
ansible-playbook -i ./hosts ./nginx.yml
```
-i option specifies the inventory file to use and where the files are located.

5. Synchronize the webpage you want to deploy
```bash
ansible-playbook -i ./hosts ./sync.yml
```


## Backend configuration
1. Update your user-name and hostname.
2. Open the terminal with the corresponding path of the repository that we are going to execute.
3. Run ubuntu with the command wsl
4. Execute Ansible: 
```bash
ansible-playbook -i ./hosts ./apache.yml
```

## Check your websites
1. To check the Frontend, open your browser and type the hostname of the public server. Example   18.170.30.71
2. To check the private server, we need to set up 
SSH connection and tunel, using a local **port** **forwarding** tunnel over an SSH connection with the next command:
```bash
ssh -L local_port:remote_private_host:remote_port user@public-hostname
```

ssh -L command is used to create a local port forwarding tunnel over an SSH connection.
-L option specifies a local port to forward from the client machine to a destination machine.


Example ssh -L 8080:10.10.10.138:80 user-name@18.170.30.71

