__<h1>How to configure HAProxy and automatic updation of it’s configuration file each time managed node is added</h1>__

![Ansible_HAProxy_HTTPD](https://miro.medium.com/max/875/1*_8s1p8MTZhFPxznYH7dzZw.jpeg)

<br>
<h2> Content </h2>
<h4><ul>
<li><i>About Ansible</i></li>
<li><i>About HAProxy</i></li>
<li><i>About Apache Httpd Webserver</i></li> 
<li><i>Project Understanding</i></li> 
</ul></h4><br>

<h2>About Ansible</h2>
<h3>What is Ansible ?</h3>
<i><b>Ansible</b> is a radically simple IT automation engine that automates cloud provisioning, configuration management, application deployment, intra-service orchestration, and many other IT needs.</i>

<h3>Why use Ansible ?</h3>
<h4><ol>
<li>
  Simple
  <ul>
    <li><i>Human readable automation</i></li>
    <li><i>No special coding skills needed</i></li>
    <li><i>Tasks executed in order</i></li>
    <li><i>Get productive quickly</i></li>
  </ul>
</li><br>
<li>
  Powerful
  <ul>
    <li><i>App deployment</i></li>
    <li><i>Configuration management</i></li>
    <li><i>Workflow orchestration</i></li>
    <li><i>Orchestrate the app lifecycle</i></li>
  </ul>
</li><br>
<li>
  Agentless
  <ul>
    <li><i>Agentless architecture</i></li>
    <li><i>Uses OpenSSH and WinRM</i></li>
    <li><i>No agents to exploit or update</i></li>
    <li><i>Predictable, reliable and secure</i></li>
  </ul>
</li><br>
</ol></h4>
For Ansible Documentation, visit the link mentioned below:<br>
https://docs.ansible.com/

<br>
<p align="center"><b>. . .</b></p><br>

<h2>About HAProxy</h2>
<ul>
  <li><i><b>HAProxy</b> is free, open source software that provides a high availability <b>load balancer</b> and <b>proxy server</b> for TCP and HTTP-based applications that spreads requests across multiple servers.</i></li>
  <li><i>It’s written in <b>C</b> and has a reputation for being fast and efficient in terms of processor and memory usage.</i></li>
</ul>

For HAProxy Documentation, visit the link mentioned below:<br>
https://www.haproxy.com/documentation/hapee/latest/onepage/intro/

<br>
<p align="center"><b>. . .</b></p><br>

<h2>About Apache HTTPD Webserver</h2>
<ul>
  <li><i>The <b>Apache HTTP Server</b>, colloquially called <b>Apache</b>, is a free and open-source cross-platform web server software, released under the terms of <b>Apache License 2.0</b>.</i></li>
  <li><i>Apache is developed and maintained by an open community of developers under the auspices of the <b>Apache Software Foundation</b>.</i></li>
  <li><i>The vast majority of Apache HTTP Server instances run on a Linux distribution, but current versions also run on Microsoft Windows, OpenVMS and a wide variety of Unix-like systems.</i></li>
</ul>

For Apache HTTPD Documentation, visit the link mentioned below:<br>
https://httpd.apache.org/docs/

<br>
<p align="center"><b>. . .</b></p><br>

<h2>Project Understanding</h2>
<p>Let's understand this implementation part by part</p>

<h3><b>Part 1 : Ansible Playbook (main.yml)</b></h3>
Let's understand it task by task:<br>

<h3><i>Common Tasks</i></h3>

<ul>
  <li>"<i><b>Creation of directory to be mounted</b></i>" : Creation of Mount Directory</li><br>
  
  ```yaml
  - name: Creation of directory to be mounted
    file:
      state: directory
      path: "{{ mount_directory }}"
  ```
  
  <br>
  <li>"<i><b>Mounting directory created to CDROM</b></i>" : Mount the directory created in previous task to the CDROM</li><br>
  
  ```yaml
  - name: Mounting directory created to CDROM
    mount:
      src: "/dev/cdrom"
      path: "{{ mount_directory }}"
      state: mounted
      fstype: "iso9660"
  ```
  
  <br>
  <li>"<i><b>YUM Repository - AppStream</b></i>" : Addition of AppStream YUM repository</li><br>
  
  ```yaml
  - name: YUM Repository - AppStream
    yum_repository:
      baseurl: "{{ mount_directory }}/AppStream"
      name: "DVD1"
      description: "YUM Repository - AppStream"
      enabled: true
      gpgcheck: no
  ```
  
  <br>
  <li>"<i><b>YUM Repository - BaseOS</b></i>" : Addition of BaseOS YUM repository</li><br>
  
   ```yaml
   - name: YUM Repository - BaseOS
     yum_repository:
       baseurl: "{{ mount_directory }}/BaseOS"
       name: "DVD2"
       description: "YUM Repository - BaseOS"
       enabled: true
       gpgcheck: no
   ```
   
   <br>
   <li>"<i><b>Stopping Firewall Service</b></i>" : Stops Firewall Service</li><br>
   
   ```yaml
   - name: Stopping Firewall Service
     service:
       name: "firewalld"
       state: stopped
       enabled: yes
   ```
   
   <br>
   <li>"<i><b>Setting SELinux Permissive</b></i>" : Sets SELinux state to permissive</li><br>
   
   ```yaml
   - name: Setting SELinux Permissive
     selinux:
       policy: targeted
       state: permissive
   ```
   
</ul>
<br><br>
<h3><i>loadbalancer (HAProxy)</i></h3>

<ul>
  <li>"<i><b>Installation of HAProxy Software</b></i>" : Installs HAProxy Software</li><br>
  
  ```yaml
  - name: Installation of HAProxy Software
    package:
      name: "haproxy"
      state: present
  ```
  
  <br>
  <li>"<i><b>HAProxy Configuration File Setup</b></i>" : Copies the configuration file from templates directory in the Controller Node to the default path of HAProxy configuration file in the Managed Node</li><br>
  
  ```yaml
  - name: HAProxy Configuration File Setup
    template:
      dest: "/etc/haproxy/haproxy.cfg"
      src: "templates/haproxy.cfg"
    notify: Restart Load Balancer
  ```
  
  <br>
  <li>"<i><b>Starting Load Balancer Service</b></i>" : Starts HAProxy Service</li><br>
  
  ```yaml
  - name: Starting Load Balancer Service
    service:
      name: "haproxy"
      state: started
      enabled: yes
  ```
  
  <br>
  <li><b>handlers</b> - "<i><b>Restart Load Balancer</b></i>" : Starts HAProxy Service</li><br>
  
  ```yaml
  - name: Restart Load Balancer
    systemd:
      name: "haproxy"
      state: restarted
      enabled: yes
  ```
  
</ul>
<br><br>
<h3><i>webserver (Apache Httpd)</i></h3>

<ul>
  <li>"<i><b>HTTPD Installation</b></i>" : Installs Apache Httpd Software only when <b>ansible_distribution</b> is RedHat</li><br>
  
  ```yaml
  - name: HTTPD Installation
    package: 
      name: "httpd"
      state: present
    when: ansible_distribution == "RedHat"
  ```
  
  <br>
  <li>"<i><b>Web Page Setup</b></i>" : Copies the web page from web directory in Controller Node to the Document Root in the Managed Node</li><br>
  
  ```yaml
  - name: Web Page Setup
    template:
      dest: "/var/www/html"
      src: "web/index.html"
  ```
  
  <br>
  <li>"<i><b>Starting HTTPD Service</b></i>" : Starts Httpd Service</li><br>
  
  ```yaml
  - name: Starting HTTPD Service
    service:
      name: "httpd"
      state: started
      enabled: yes
  ```
  
</ul>
<br><br>
<h3><i>Complete Playbook</i></h3>

```yaml
- hosts: loadbalancer
  vars_files:
    - vars.yml
  tasks:
  - name: Creation of directory to be mounted
    file:
      state: directory
      path: "{{ mount_directory }}"
      
  - name: Mounting directory created to CDROM
    mount:
      src: "/dev/cdrom"
      path: "{{ mount_directory }}"
      state: mounted
      fstype: "iso9660"
      
  - name: YUM Repository - AppStream
    yum_repository:
      baseurl: "{{ mount_directory }}/AppStream"
      name: "DVD1"
      description: "YUM Repository - AppStream"
      enabled: true
      gpgcheck: no
      
  - name: YUM Repository - BaseOS
    yum_repository:
      baseurl: "{{ mount_directory }}/BaseOS"
      name: "DVD2"
      description: "YUM Repository - BaseOS"
      enabled: true
      gpgcheck: no
      
  - name: Installation of HAProxy Software
    package:
      name: "haproxy"
      state: present
      
  - name: HAProxy Configuration File Setup
    template:
      dest: "/etc/haproxy/haproxy.cfg"
      src: "templates/haproxy.cfg"
    notify: Restart Load Balancer
    
  - name: Starting Load Balancer Service
    service:
      name: "haproxy"
      state: started
      enabled: yes
      
  - name: Stopping Firewall Service
    service:
      name: "firewalld"
      state: stopped
      enabled: yes
      
  - name: Setting SELinux Permissive
    selinux:
      policy: targeted
      state: permissive
      
handlers:
  - name: Restart Load Balancer
    systemd:
      name: "haproxy"
      state: restarted
      enabled: yes
      
      
- hosts: webserver
  vars_files:
    - vars.yml
  tasks:
  - name: Creation of directory to be mounted
    file:
      state: directory
      path: "{{ mount_directory }}"
      
  - name: Mounting directory created to CDROM
    mount:
      src: "/dev/cdrom"
      path: "{{ mount_directory }}"
      state: mounted
      fstype: "iso9660"
      
  - name: YUM Repository - AppStream
    yum_repository:
      baseurl: "{{ mount_directory }}/AppStream"
      name: "DVD1"
      description: "YUM Repository - AppStream"
      gpgcheck: no
      
  - name: YUM Repository - BaseOS
    yum_repository:
      baseurl: "{{ mount_directory }}/BaseOS"
      name: "DVD2"
      description: "YUM Repository - BaseOS"
      gpgcheck: no
      
  - name: HTTPD Installation
    package: 
      name: "httpd"
      state: present
    when: ansible_distribution == "RedHat"
    
  - name: Web Page Setup
    template:
      dest: "/var/www/html"
      src: "web/index.html"
      
  - name: Setting SELinux Permissive
    selinux:
      policy: targeted
      state: permissive
       
  - name: Stopping Firewall Service
    service:
      name: "firewalld"
      state: stopped
      enabled: yes
      
  - name: Starting HTTPD Service
    service:
      name: "httpd"
      state: started
      enabled: yes
```

<br><br>
<hr style="border:2px solid gray"> </hr>
<h3><b>Part 2 : Variable Files (vars.yml)</b></h3>

```yaml
  mount_directory: "/dvd1"
```

<br><br>
<hr style="border:2px solid gray"> </hr>
<h3><b>Part 3 : Web Page (index.html)</b></h3>

```html
  <body bgcolor='aqua'>
  <br><br>
  <b>IP Addresses</b> : "{{ ansible_all_ipv4_addresses }}"
```

<p>Specifies Web Page that is accessed from Load Balancer’s IP Address</p>
<br><br>
<hr style="border:2px solid gray"> </hr>
<h3><b>Part 4 : HAProxy Configuration File Template (haproxy.cfg)</b></h3>

```
#---------------------------------------------------------------------
# Example configuration for a possible web application.  See the
# full configuration options online.
#
#   https://www.haproxy.org/download/1.8/doc/configuration.txt
#
#---------------------------------------------------------------------
#---------------------------------------------------------------------
# Global settings
#---------------------------------------------------------------------
global
    # to have these messages end up in /var/log/haproxy.log you will
    # need to:
    #
    # 1) configure syslog to accept network log events.  This is done
    #    by adding the '-r' option to the SYSLOGD_OPTIONS in
    #    /etc/sysconfig/syslog
    #
    # 2) configure local2 events to go to the /var/log/haproxy.log
    #   file. A line like the following can be added to
    #   /etc/sysconfig/syslog
    #
    #    local2.*                       /var/log/haproxy.log
    #
    log         127.0.0.1 local2
chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon
# turn on stats unix socket
    stats socket /var/lib/haproxy/stats
# utilize system-wide crypto-policies
    ssl-default-bind-ciphers PROFILE=SYSTEM
    ssl-default-server-ciphers PROFILE=SYSTEM
#---------------------------------------------------------------------
# common defaults that all the 'listen' and 'backend' sections will
# use if not designated in their block
#---------------------------------------------------------------------
defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
    option http-server-close
    option forwardfor       except 127.0.0.0/8
    option                  redispatch
    retries                 3
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    timeout client          1m
    timeout server          1m
    timeout http-keep-alive 10s
    timeout check           10s
    maxconn                 3000
#---------------------------------------------------------------------
# main frontend which proxys to the backends
#---------------------------------------------------------------------
frontend main
    bind *:8080
    acl url_static       path_beg       -i /static /images /javascript /stylesheets
    acl url_static       path_end       -i .jpg .gif .png .css .js
use_backend static          if url_static
    default_backend             app
#---------------------------------------------------------------------
# static backend for serving up images, stylesheets and such
#---------------------------------------------------------------------
backend static
    balance     roundrobin
    server      static 127.0.0.1:4331 check
#---------------------------------------------------------------------
# round robin balancing between the various backends
#---------------------------------------------------------------------
backend app
    balance          roundrobin

{%  for  hosts  in groups['webserver']  %}
    server  app1 {{ hosts }}:80 check
{% endfor %}
```

<p><b>Jinja template</b> and <b>for</b> loop specified above makes HAProxy configuration file more dynamic as it would add managed nodes specified under <b>‘webserver’</b> host dynamically.</p>

<br><br>
<hr style="border:2px solid gray"> </hr>
<h3><b>Part 5 : Output</b></h3>

<h3><i>loadbalancer</i></h3>

![haproxy](https://miro.medium.com/max/875/1*4eqCEbjDnQO6oLpvR8-1mQ.png)

<p align="center"><b>HAProxy Configuration File after Ansible Playbook Execution-1</b></p><br><br>

![haproxy](https://miro.medium.com/max/875/1*Ca_HhBeqXf92SRgyauLx2A.png)

<p align="center"><b>HAProxy Configuration File after Ansible Playbook Execution-2</b></p><br><br>

![haproxy](https://miro.medium.com/max/875/1*VUKKrmo0mRoB0MkkVC4GKQ.png)

<p align="center"><b>HAProxy Configuration File after Ansible Playbook Execution-3</b></p><br><br>

<h3><i>webserver</i></h3>

![webserver](https://miro.medium.com/max/875/1*Z7cGOYkS9K-9iIle3BsMRA.png)

<p align="center"><b>Webserver 1</b></p><br><br>

![webserver](https://miro.medium.com/max/875/1*xWGUGIrNDCuCAjNB4MgKJA.png)

<p align="center"><b>Webserver 2</b></p><br><br>

<h3><i>Output GIF</i></h3>

![output_gif](https://miro.medium.com/max/875/1*Bj1xkciW4OJ_Sxq_0v_mqQ.gif)


<br>
<p align="center"><b>. . .</b></p><br>

<h2>Thank You :smiley:<h2>
<h3>LinkedIn Profile</h3>
https://www.linkedin.com/in/satyam-singh-95a266182





