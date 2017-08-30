# Web Server Setup: A Descriptive Guide


*Per the title, this is meant to be a ***descriptive*** (albeit ***nonexhaustive***)*  
*guide to setting up ***Apache*** and ***Nginx***, the two most popular*  
*web servers.  I decided to collate my notes in the hopes of establishing an*  
*overall rationalization of setup steps rather than a checklist which*  
*can easily sourced elsewhere.  If you're looking for a mere checklist, read*  
*no further.*  

<div id="toc">
<p id="toc-header">Contents</p>
	<ol>
		<li><a class="toc-link" href="#background">Background</a></li> 
		<li><a class="toc-link" href="#overview">Overview: Apache & Nginx</a></li> 
		<li><a class="toc-link" href="#summary-comps">Summary Comparison</a></li> 
		<li><a class="toc-link" href="#setup">Setup</a></li>
			<ul>
				<li><a class="toc-link" href="#installation">Installation</a></li>
				<li><a class="toc-link" href="#global-cfg">Set Global Configurations</a></li>
				<li><a class="toc-link" href="#virtual-hosts">Setup Virtual Hosts</a></li>
				<li><a class="toc-link" href="#wsgi">WSGI Applications</a></li>
				<li><a class="toc-link" href="#venv">Virtual Environments</a></li>
				<li><a class="toc-link" href="#firewalls">Firewall Settings</a></li>
				<li><a class="toc-link" href="#cfg-activate">Activating/Reloading Configurations</a></li>
			</ul>
		<li><a class="toc-link" href="#conclusion">Conclusion</a></li>
	</ol>
</div>

<a id="background"></a>
## Background

<a class="toc-link" href="#toc">(Back to Contents)</a>

Thanks to the growth of web-based applications and the proliferation  
of VPS/private cloud providers, there is no shortage of documentation  
on setting up web servers, the software which handles incoming http requests  
from clients.  

From firewall settings to Unix socket management, there is a step-by-step  
manual out there to guide you every step of the way on what was once largely  
relegated to the domain of the mysterious and reclusive sysadmin.  Some VPS  
providers have been especially active on this front, motivated by the prospect  
of increased subscriber numbers/accounts via the elucidation of these once  
arcane steps.  

Adding to this literature is the seemingly endless regurgitation of these  
steps in blog posts for which the vast majority appear to be ***blogging for***  
***blogging's sake*** rather than any insightful commentary on the purpose or  
meaning behind the steps involved.  

To be sure, the task at hand does indeed require a list of tasks to be  
performed and it's helpful to maintain checklists for these along the way.  
However, I can't help but feel that one might be better served by an  
understanding of the *reasoning* behind these key steps.  Grasping the  
rationale of the setup in this way may help avoid redundant cross-checks  
of the steps it appears many of the list-only guides appear to be serving.  

As such, I offer below a summary web server setup *alternative guide* (a verbose  
commentary-filled read which will hopefully facilitate key concept retention  
better than performing a rote list of steps) for Apache and Nginx, the two most  
popular web servers.  


<a id="overview"></a>
## Overview

<a class="toc-link" href="#toc">(Back to Contents)</a>

To put things in context for the descriptive steps ahead, it helps to have  
a little background knowledge about these web servers which I present here  
in broad brush strokes (skip at will, if you are already familiar with    
background information on these web servers).  


### Apache

Written by Robert McCool after splitting from the [NCSA HTTPd project](https://en.wikipedia.org/wiki/NCSA_HTTPd),  
Apache was released in 1995 and underwent a major upgrade in 2002 to  
accommodate multithreading and filtering and a license upgrade in 2004 to  
the Version 2.0 Apache Software Foundation license.  Thus, all processes,  
directory names, etc. are called 'apache2', (not 'apache').  

Apache evolved in tandem with the *Common Gateway Interface* or CGI (itself  
authored in part by Apache's creator), the original protocol which standardized  
the communication between web server software (like Apache) and webpage  
generating scripts like Perl (at the time) and the likes of PHP, Python today.  
Its architecture consists of a small core to handle http request/response  
processing surrounded by a satellite of modules ('mods') to extend this core  
functionality.  

One key advantage is that a language processor of choice can be embedded within  
Apache by installing modules like mod\_python which means execution of dynamic  
content can take place internally within the web server without the need for any  
further configuration.  This process can be made yet more efficient by way of  
installing other complementary third-party modules like *mod\_wsgi*  
(the Web Server Gateway Interface for Python, as an example) which standardize  
scripting language - web server communication.  

Apache's traditional architecture to answer http requests involved spawning new  
processes which meant resource consumption increased with the number of  
connections made to the server, in contrast with the Nginx approach  
(see below).  The development of FastCGI, an alternative to CGI which keeps a  
persistent process running that itself handles tasks via multiple threads (thus  
dramatically lowering the process-spawning overhead associated with pure CGI)  
has mitigated, though not entirely eliminated, this performance drawback.  

Version 2.4 introduced the Event MultiProcessing Module (MPM) to fix  
performance bottlenecks when serving static pages.  Nevertheless, Apache  
is still perceived to be better at handling server-heavy tasks (such as  
those involving database transactions or calls to server-side routines in the  
application) and thus better suited for handling dynamic websites.  


### Nginx

Developed in 2002/released in 2004 by the Russian Igor Sysoev, Nginx rose  
to the challenge of solving the 10k concurrent connection problem which was  
a limitation many web servers had at the time (including Apache).  Nginx was  
built specifically with the intention of beating Apache at its own game.  

In contrast to Apache, Nginx's architecture is one of an asynchronous event  
driven model in which individual worker processes can handle thousands of  
threads by way of a rapid looping process which checks for incoming requests  
and processes them as it catches them.  

Events (connections) within the loop are served asynchronously in a  
non-blocking manner and then removed when the connection is closed.  
The result is a small memory footprint and more predictable resource  
consumption which translates to easier scalability as compared with  
Apache.  

The Python WSGI standard proved so successful that it spawned the popular  
server container *uWSGI*, itself written in C, which acts as a universal  
Python interpreter for a variety of application languages and processes  
incoming requests by invoking the Python-equivalent callable.  

Nginx boasts native support for *uwsgi*, the very fast binary communication  
protocol of uWSGI.  While Apache modules supporting uwsgi do exist, they are  
perceived to be [awkwardly implemented](http://uwsgi-docs.readthedocs.io/en/latest/Apache.html), in comparison.  

Speed aside, given that Apache modules can run scripting languages, a lot  
of the heavy lifting for server-side back-end applications (thus dynamic  
sites) tends to be delegated to Apache whereas Nginx is perceived to be the  
better choice for serving static contents at scale (and speed).  In fact, many  
Nginx configurations hand off such server-side tasks to other processes and/or  
Apache itself.  

The traditional rivalry between these two web servers has evolved in  
recent years to a complementary approach wherein Nginx may well serve as a  
reverse proxy for and/or load balancer in front of Apache.  Nginx themselves  
expresses the desirability for this complementary approach in their  
[own documentation](https://www.nginx.com/blog/nginx-vs-apache-our-view/).

<a id="summary-comps"></a>
#### Summary Comparison:

<a class="toc-link" href="#toc">(Back to Contents)</a>

***Apache:***    
* most popular (40-50% of websites, depending on poll)
* multi-process architecture is resource-intensive
* satellite modules for extensibility (including FastCGI)
* performance drag with many connections for static sites
* good for server-side heavy (dynamic) applications    


***Nginx:***  
* second most popular (20-30% of websites, depending on poll)
* event driven aysnch model means low memory footprint
* FastCGI, WSGI and uWSGI-native
* higher performance serving static than Apache
* hands off server-heavy transactions to other processes (incl Apache)

<a id="overview"></a>
## Setup
<a class="toc-link" href="#toc">(Back to Contents)</a>

*Note: to simplify things, I assume an Ubuntu Linux distribution for the steps*    
*which follow.  (Naturally, the choice of OS will impact configuration options*  
*but I have tried to keep the following commentary as OS-agnostic as possible,*  
*except where noted):*  

</br>

<a id="installation"></a>
### 1). Install Apache and Nginx:  
<a class="toc-link" href="#toc">(Back to Contents)</a>


<pre class="snippet">
sudo apt-get install apache2
sudo apt-get install nginx
</pre>


Enough said.  

</br>

<a id="global-cfg"></a>
### 2). Set Global Configurations:  
<a class="toc-link" href="#toc">(Back to Contents)</a>

As a first step, we need to set some high-level configurations for both Apache  
and Nginx.  Despite being very top-of-the-trees, they are crucially important.  
For instance, we need to specify the directory for error logs.  If we don't  
know the location of the error logs, we won't be able to track down errors and  
debug issues if and when they crop up.  

Assuming the web servers were installed using default settings, the *global*  
configuration files for each web server will be the following files:

<pre class="snippet">
/etc/apache2/apache2.conf
/etc/nginx/nginx.conf
</pre>

*(Recall the '/etc' directory is the applications configuration directory*    
*in Linux systems.  Also note that modifications to configurations files here*  
*typically require admin/superuser privileges, thus on Ubuntu you'll need to*  
*prefix editing commands with 'sudo').*  

While these files differ in their structure and content, they conveniently  
allow for configurations to be specified in space separated key-value couplings  
as in:

<pre class="snippet">
[parameter] [argument]
</pre>

Both Apache and Nginx allow for the *include* directive which means that  
further configurations can be loaded as sourced from the indicated  
directory and/or file argument following the include keyword, as in:

<pre class="snippet">
include /path/to/my/dir/or/file
</pre>

For *many* (though not necessarily *most*) use-case scenarios the default  
settings in these global configuration files tend to serve just fine the way  
they are.  Between the two of them, the apache2.conf file is heavily commented  
right out of the box; thus, plenty of guidance is available in these comments  
for various settings (nginx.conf comments are more terse in comparison, but  
this is mitigated by the self-explanatory names of the directives in it).  

The operative attribute of the apache2.conf file is that it acts like a spine  
which includes other apache configuration files, namely: ports, mods-enabled,  
conf-enabled, and sites-enabled configuration files (more on these later).  
Directives in the nginx.conf global configurations file, by contrast, come  
grouped in three sections, which we'll also look at more closely.

<br></br>
### Global Configurations:  Common Concepts

Ignoring comments and their default listed order, some important configuration  
settings in the **apache2.conf** file are as follows:

<pre class="snippet">
User ${APACHE_RUN_USER}
Group ${APACHE_RUN_GROUP}
PidFile ${APACHE_PID_FILE}
ErrorLog ${APACHE_LOG_DIR}/error.log

IncludeOptional conf-enabled/*.conf
IncludeOptional sites-enabled/*.conf

KeepAlive On
KeepAliveTimeout 5
</pre>


Note the similarity to their counterparts in the **nginx.conf** file:

<pre class="snippet">
user www-data;
pid /run/nginx.pid;
access_log /var/log/nginx/access.log;
error_log /var/log/nginx/error.log;

include /etc/nginx/conf.d/*.conf;
include /etc/nginx/sites-enabled/*

keepalive_timeout 65;

</pre>

As can be seen from these global settings, both Apache and Nginx allow values  
for user run-profiles to be set here.  The Apache settings in curly brackets  
are environment variables that were established during installation but the  
parameters are similar (indeed, the default Apache user is even 'www-data', per  
the default entry in the ngfinx.conf file).

Both files allow for configuring where the process id file is to be written  
which is useful for monitoring purposes and the access and error log locations  
may also be set here.  While there is no access log file configuration in the  
apache2.conf file, the default location is merely the apache2 counterpart  
to the setting in the nginx.conf file, namely: **/var/log/apache2/access.log**.  

**Available vs. Enabled**

In both the Apache and the Nginx ecosystems, sites and configurations must  
first be made available before they can be enabled.  What this means is that  
both web servers take advantage of *nix environments wherein symbolic links  
to physical directories/files determine what components are made 'active' or  
'enabled'.  

Thus, a configuration for a certain site must reside in a directory  
(conventionally annotated as 'available' in the web servers' parlance)  
but the site is not made active unless a symbolic link pointing to it is  
also included in the directory annotated as 'enabled'.  

This is a convenient way for site administrators to control what sites or  
configurations they wish to make active (enabled).  By simply creating or  
removing a symbolic link, they can switch the site/configuration's status  
on or off without having to delete/modify the underlying files/directories.  

The 'include' and 'IncludeOptional' directives in the global configurations  
file specifies the location of the enabled configuration and sites directories.  

**Virtual Hosts**

The concept of a *Virtual Host* is another characteristic these two web servers  
have in common.  Similar in concept to *virtual environments*, virtual hosts,  
configured by virtual host files, allow both Apache and Nginx to be configured  
for multiple sites which may require separate configurations. 

These are the meat and potatoes of site configurations for both servers which  
will require more delving into but for now note that the configuration files  
for virtual hosts reside *sites-available* sub-directories for each server (and  
also the *sites-enabled* sub-directory as symlinks if they are enabled/active).  

Now that we've seen the common configuration directives between the two web  
servers, let's have a look at what makes them different.

*(Note: I only touch upon what I consider to be ***salient*** characteristics*  
*of each web server's setup, not every detail.  As such, the usual caveat*  
*applies here: please refer to the official documentation for more information)*  

<br></br>
#### Global Configurations: Disparate Concepts

***Apache-Specific:***

<pre class="snippet">
Include ports.conf
IncludeOptional mods-enabled/*.load
IncludeOptional mods-enabled/*.conf

AccessFileName .htaccess

Timeout 300
MaxKeepAliveRequests 100

Mutex file:${APACHE_LOCK_DIR} default
HostnameLookups Off
LogLevel warn
</pre>

As noted in the background section, Apache's core functionality is extended by  
modules or 'mods.'  A default set of modules comes with the Apache installation  
but other non-standard/custom modules may be installed to further extend  
functionality.  These reside in the mods-available and mods-enabled child  
directories.  The *.load files govern how the module is loaded into Apache  
while the *.conf files specify how the modules are to be used within it.  

The *.htaccess* files is a somewhat outdated configuration file that enables    
configurations changes to be made to virtual hosts for setups in which  
site administrators may not have root access on the server system to edit  
the virtual host configuration file directly.  

Apache advises against use of the .htaccess file but it should be noted  
that it *can* exercise an override for uses such as permissioning if  
configured to do so.  Since it overrides the host configuration files, it  
should be used with care (if at all).  There is no such equivalent in Nginx.  

A key setting we should pay attention to is the ports configuration.  
From the outset, Apache requires that we specify what ports it is meant  
to listen on for HTTP connections.  This is specified in the:

<pre class="snippet">
/etc/apache2/ports.conf
</pre>

file and is by default port 80 (for HTTP).  If you are pursuing a non-standard  
setup which might require another port entry, such as perhaps a simultaneous  
installation of Nginx which does not act as a reverse proxy for Apache, you may  
need to edit this so that it listens on a different port from the one Nginx is  
configured to listen on.  

Miscellaneous other settings hint at some of Apache's limitations as  
a web server, specifically with regard to its multi-process (and blocking)  
architecture.  For instance, the *KeepAlive*-related settings are used to  
configure keeping open a given TCP connection between Apache and a remote  
client to improve efficiency and to override the default behavior of an  
HTTP connection to close upon transfer of a file.  

However, there is a limit to how many of these may be kept open after which the  
server would not be able to accept any more HTTP connections. A good description  
of this may be found [here](https://www.nginx.com/blog/http-keepalives-and-web-performance/).

<br></br>
***Nginx-Specific:***

<pre class="snippet">
--first section:
worker_processes auto;

--events section:
events {
        worker_connections 768;
        # multi_accept on;
}

http {
		...;
		server_names_hash_bucket_size 64;
		...;
		include /etc/nginx/mime.types;
		...;
}
</pre>

As mentioned before, the nginx.conf file comes in three sections and these  
Nginx-only directives stand out (again, refer to the official documentation  
for information on the other settings).  

The worker-process variable specifies how many simultaneous threads of nginx  
to run.  Nginx's own documentation says this should correspond to the number of  
cores available on the server.  Setting this to 'auto' will instruct Nginx  
to fetch this information and set this variable automatically.  

The events variable is a reference to incoming/client-generated traffic.  
You can set the maximum number of worker connections in this section as well  
as permission a worker process to accept multiple connections in the  
multi_accept parameter.

The hash bucket size parameter is generally commented but it may be worthwhile  
to *uncomment* this parameter so as to avoid running into hash memory bucket  
issues.  This is just the memory allocated by Nginx to look up the servers  
in a hash table it uses for this purpose.  

Finally, the mime.types file setting is a path to a file which by default  
is itself replete with many file format settings.  Since these run the gamut  
from txt, html, css to gzip and others, there's a fair chance that it will  
include many file format requirements but may be referenced/edited to ensure  
it meets your very specific needs.  

<a id="virtual-hosts"></a>
### 3). Setup Virtual Hosts:

<a class="toc-link" href="#toc">(Back to Contents)</a>

Both Apache and Nginx come with default virtual host files which may be used  
as templates for setting up your own virtual hosts.  To repeat, a virtual host  
file is the configuration file which defines your sites' particulars, such the  
port it should be served on the server, what the domain name is, and the like.  
The sample host files are heavily commented and the recommended practice is to  
copy the default sample files to use as boilerplates to modify as you desire.

The files in question are:

<pre class="snippet">
/etc/apache2/sites-available/000-default.conf
/etc/nginx/sites-available/default
</pre>

Let's have a look at these without the comments.

***Apache:***
<pre class="snippet">
&lt;VirtualHost *:80>
    ServerName www.example.com
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
&lt;/VirtualHost>
</pre>

***Nginx:***
<pre class="snippet">
server {
        listen 80 default_server;
        listen [::]:80 default_server;
        root /var/www/html;        
        index index.html 
        server_name _;

        location / {                
                try_files $uri $uri/ =404;
        }
</pre>

Here again the two web servers share some commonality in the configuration of  
virtual hosts.  As can be seen from the two files, configurations which set  
the server (hostname) and document root serve the same function in both cases.  

Ports to listen to for the specific host are also set in these files.  

For **Apache** this happens in the line:
<pre class="snippet">
&lt;VirtualHost *:80>
</pre>

whereas for **Nginx** this is configured in the line:
<pre class="snippet">
listen 80 &lt;...&gt;;
</pre>

By default, both Apache and Nginx expect website directories to be housed  
in the '/var/www' or 'var/www/html' directory.  Indeed, default installations  
of each web server will result in a generic "index.html" file being placed  
in the latter so that when the web server is activated and configurations are  
read, a the corresponding Apache or Nginx welcome page will appear in the  
browser, if the web server has been set up successfully.  

The ability to specify virtual hosts for both Apache and Nginx means that  
multiple domains/sites can be hosted by one the same server with each  
host having its own configuration file.

For instance, the relevant Apache entries would look something like:

<pre class="snippet">
ServerName www.apache1.com    
DocumentRoot /var/www/apache1.com

ServerName www.apache2.com    
DocumentRoot /var/www/apache2.com    
</pre>

Similarly, Nginx entries would look something like:

<pre class="snippet">
root /var/www/ngix1.com;        
index index.html;
server_name www.ngix1.com;

root /var/www/nginx2.com;
index index.html;
server_name www.ngix2.com;
</pre>

The default document root directories need not be used for host sites;  
however, alternative locations should be chosen carefully as the original  
intent was to disallow access to the root filesystem outside of these  
directories for security reasons to prevent, for instance, the likes of  
[directory traversal attacks](https://en.wikipedia.org/wiki/Directory_traversal_attack).

As can be seen from these configurations, the document root directory is  
configured directly within the host files (which would override any other  
document root directories specified elsewhere, as with Apache global  
configurations).

For Apache host configurations serving files outside of the default directory  
requires an *alias* entry along with permissioning to override the default access  
settings, like so:

<pre class="snippet">
Alias   "/otherRoot/" "/otherRoot/path/to/my/project/"

&lt;Directory "/otherRoot/path/to/my/project/">
    Order allow,deny  
	Allow from all 
    Require all granted
&lt;/Directory>
</pre>

*(Note: for versions of Apache 2.4 or above, the directory permissions syntax*  
*changed from 'Order allow,deny Allow from all' to 'Require all granted').*

One Nginx-specific host file configuration is the entry:  

<pre class="snippet">
listen 80 default_server;
</pre>


This is meant to serve as the fallback/default server name in the event  
that a client connection should be attempted to an invalid endpoint on the  
server.  This is more a legacy configuration as nowadays clients are likely  
to attempt connections using names, meaning such failures are likely to be  
extremely rare.


<a id="wsgi"></a>
### 4). WSGI Applications:

<a class="toc-link" href="#toc">(Back to Contents)</a>

*At this point, I would like to digress a bit from my application-agnostic*  
*approach to include a few tips on setup-related issues pertaining to*  
*Python-specific web applications.  Full disclosure: the need for me to detail*  
*these arises from my own Python-centric development tendency.*

If you're serving a Python-based or other WSGI-protocol compliant application,  
some modifications to the default setup steps described above are in order.  

***Apache***

For the Apache web server, the *mod_wsgi* module must first be installed,  
ergo:

<pre class="snippet">
apt-get install libapache2-mod-wsgi
</pre>

Next, a *.wsgi file must be created in the project directory.  This file will  
be the entry point into the web application/site-project.  Thus, a sample wsgi  
file called 'controller.wsgi' might have the following full path:

<pre class="snippet">
/var/www/www.apache1.com/controller.wsgi
</pre>

Edit this file so that the top of it reads as follows:

<pre class="snippet">
import sys
sys.path.insert(0, '/var/www/www.apache1.com')
</pre>

( where /var/www/www.apache1.com is the path to the project folder ).

The Apache host file configuration would also require modification to add the  
following lines:

<pre class="snippet">
&lt;VirtualHost:*80>
WSGIDaemonProcess appName user=user1 group=group1 threads=5
WSGIScriptAlias / /var/www/www.apache1.com/controller.wsgi
&lt;/VirtualHost>
</pre>


where appName is the name of the application which must be run in order to  
serve the site.  


***Nginx***

The modifications required to run a WSGI application in Nginx differ from those  
in Apache server.  

Recall from the introduction that Nginx has native support for the uwsgi wire  
communication protocol.  However, the project environment (whether virtual or  
physical) must also have uwsgi installed.

For Python-based applications, this can be installed using the pip installer:  

<pre class="snippet">
pip install uwsgi
</pre>

A quick test to serve the application on localhost (port 5000) from the command  
line may then be run as follows:

<pre class="snippet">
uwsgi --socket 0.0.0.0:5000 --protocol=http -w wsgi:app
</pre>

where app is the name of the application (without extension).  If the project  
is in a virtual environment, the environment should first be activated before  
running the above command.

Once this is working, we will need to create a uWSGI configuration file for use  
with Nginx.  Like the sample *.wsgi file per our Apache example, this file will  
reside in our project directory but with an *.ini extension as this will later  
be used as an argument into a systemd file which is responsible for launching  
processes upon server reboots.

A sample ini file may look as follows:

<pre class="snippet">
[uwsgi]
module = wsgi:appName
master = true
processes = 5
socket = www.nginx1.com.sock
chmod-socket = 660
vacuum = true
die-on-term = true
</pre>

where the header signifies the file is a *uwsgi* configuration files and  
appName is the name of the application serving the site.  

The remaining specs instruct Nginx to launch a master process to supervise  
worker processes spawned, processes indicates the number of processes to spawn,  
socket is the name of the Unix socket the uwsgi protocol is instructed to  
create/use which is subsequently permissioned for use with chmod.  Finally,  
the parameters vacuum means to clean up on closed/interrupted connection and  
die-on-term means to kill the process if manually terminated, per expected  
behavior.  

A systemd service file (nginx1.service) file may then be created as follows:  

<pre class="snippet">
[Unit]
Description=uWSGI instance to serve nginx1 project
After=network.target
[Service]
User=<user>
Group=www-data
WorkingDirectory=/var/www/www.nginx1.com
Environment="PATH=/var/www/www.nginx1.com/virtualenv/bin"
ExecStart=/var/www/www.nginx1.com/virtualenv/bin/uwsgi --ini controller.ini
[Install]
WantedBy=multi-user.target
</pre>

One helpful resource which details the use of Python-based web applications  
and their uWSGI configuration requirements for Nginx can be found [here](https://www.digitalocean.com/community/tutorials/how-to-serve-flask-applications-with-uwsgi-and-nginx-on-ubuntu-16-04).

Finally, the corresponding uwsgi entries will need to be added to the Nginx  
virtual hosts file to ensure that Nginx serves our application through the  
uWSGI entry point.  

Specifically, note that we need to further qualify the **location setting** in our  
Nginx host file to read along the lines of the following:  

<pre class="snippet">
server {
	...;
	...;
	location / {
	    include uwsgi_params;
	    uwsgi_pass unix:/var/www/www.nginx1.com/www.nginx1.com.sock;
	}
}
</pre>

This invokes the uwsgi parameters and specifies the socket connection over  
which Nginx should communicate with the application.


<a id="venv"></a>
### 5). Virtual Environments:

<a class="toc-link" href="#toc">(Back to Contents)</a>

If your application folder has been setup to run in a virtual environment,  
then still more modifications need to be made to these setups.

For Apache, these two extra lines are required at the top of our wsgi file:

<pre class="snippet">
activate_this = '/var/www/www.apache1.com/virtualEnvDir/bin/activate_this.py' 
execfile(activate_this, dict(__file__=activate_this)) 
</pre>

This is required so that Apache executes code within the virtual environment  
rather than in the physical/non-virtual project path where one or more required  
components may be missing for functionality of the site.  

Under Nginx, note that our \*.service file sets the *Environment* variable to the  
virtual environment path in the www.nginx1.com directory:

<pre class="snippet">
...
Environment="PATH=/var/www/www.nginx1.com/virtualenv/bin"
...
</pre>

thus ensuring the application runs from within the virtual environment setup.

<a id="firewalls"></a>
### 6). Firewall Settings:

<a class="toc-link" href="#toc">(Back to Contents)</a>

With our Apache and Nginx global configurations set, our virtual hosts files  
configured, and WSGI-specific and virtual environment settings/configurations  
in place, we have to ensure our firewall settings are correctly permissioned.

Under Ubuntu distributions, this is where the 'ufw' utility comes in handy.  
Per some of the other setup steps reviewed thus far, changing firewall settings  
will require super user privileges, hence the 'sudo' prefix for the following  
commands.

The command is simple and is as follows:
<pre class="snippet">
sudo ufw allow <port>
</pre>

If our requirements are that we must allow incoming traffic on ports 80 and  
8080, for instance, then we would simply run the following commands:

<pre class="snippet">
sudo ufw allow 80
sudo ufw allow 8080
</pre>

Of course, this assumes our firewall is enabled.  If not, *take care to ensure*  
*that it is configured with all the correct settings ***before enabling****  so as to  
avoid undesirable outcomes like being locked out of your own server(!)  

A shorthand available for Nginx ufw use is:

<pre class="snippet">
sudo ufw allow 'Nginx Full'
</pre>

This will enable Nginx for both HTTP and HTTPS (the secure/encrypted protocol  
we have *not* touched upon here).

To check that your ports are open, you can run the following command:

<pre class="snippet">
sudo ufw status
</pre>

to see a list of active ports.

For more on using Nginx as a reverse proxy for Apache (rather than merely  
listening on different ports), [here](https://lukearmstrong.github.io/2016/12/use-nginx-apache-at-the-same-time/) is a good article for that.

<a id="cfg-activate"></a>
### 7). Activating/Reloading Configurations:

<a class="toc-link" href="#toc">(Back to Contents)</a>

Bear in mind that whenever a change is made to a configuration file or a host  
file is added, configurations will need to be re-read by the corresponding  
web server.  Sometimes this may mean stopping/restarting the web server while  
other times it might mean just forcing configurations to be reloaded.

Both Apache and Nginx come with a set of very handy utilities for these.  
As with other facets of these two servers, these also resemble each other.

Some utilities for Apache are as follows:

<pre class="snippet">
sudo a2ensite www.apache1.com
sudo a2dissite www.apache1.com
sudo systemctl start apache2.service
sudo systemctl stop apache2.service
sudo systemctl restart apache2.service
</pre>

These commands enable (creating a symlink in enabled directory), disable,  
start, stop, and restart Apache respectively.

The Nginx equivalents are:

<pre class="snippet">
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx
sudo systemctl reload nginx
sudo systemctl disable nginx
</pre>

Similarly, these commands tart, stop, restart, reload, and disable Nginx.  
Enabling/disabling is a manual process of creating/destroying symbolic links.


<a id="conclusion"></a>
## Conclusion

<a class="toc-link" href="#toc">(Back to Contents)</a>

Despite the disparate architecture of the Apache and Nginx web servers, I have  
attempted to present here the overlapping set of configuration requirements  
involved in setting up each web server.  I have also attempted to contrast the    
important respects in which their setup requirements diverge from each other.  

In so doing, I have attempted to convey a rationalization of the setup steps  
involved - taken as a whole - in order to facilitate *key concept retention*  
better than might otherwise be possible when following a mere checklist of  
steps alone.  

I hope you find this descriptive guide to be as helpful to you as it was to me.  

For official documentation on these two web servers, see: [Apache](https://httpd.apache.org/) and [Nginx](http://nginx.org/).

