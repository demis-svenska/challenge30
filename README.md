

# PART 1.1
# install python,django,celery,uwsgi,nginx on ubuntu 16.04 LTS


## install python, pip
-------------------------

	sudo apt-get update 
	sudo apt-get install python-pip 
	sudo -H pip install --upgrade pip 

## install virtualenv
-------------------------------------------

	sudo -H pip install virtualenv virtualenvwrapper 
	echo "export WORKON_HOME=~/Env" >> ~/.bashrc 
	echo "source /usr/local/bin/virtualenvwrapper.sh" >> ~/.bashrc 
	source ~/.bashrc

## install django and create first site
----------------------

	mkvirtualenv firstsite  
	pip install django 
	pip install celery  
	cd ~ 
	django-admin.py startproject firstsite 
	cd ~/firstsite 
	~/firstsite/manage.py migrate 
	~/firstsite/manage.py createsuperuser 
		username: admin 
		password:Pfix0insa 
nano ~/firstsite/firstsite/settings.py 

	ALLOWED_HOSTS = ['your-ip-address', 'localhost'] and 
	add STATIC_ROOT = os.path.join(BASE_DIR, 'static/') 
	
~/firstsite/manage.py collectstatic 

test running:
	
	~/firstsite/manage.py runserver 0.0.0.0:8080

## install uwsgi
-----------------------------------------

	sudo -H pip install uwsgi	
	sudo mkdir -p /etc/uwsgi/sites 
	sudo nano /etc/uwsgi/sites/firstsite.ini 

		[uwsgi]
		project = firstsite
		uid = $USER
		base = /home/%(uid)
		chdir = %(base)/%(project)
		home = %(base)/Env/%(project)
		module = %(project).wsgi:application

		master = true
		processes = 5

		socket = /run/uwsgi/%(project).sock
		chown-socket = %(uid):www-data
		chmod-socket = 660
		vacuum = true

 Note! replace $USER with username  

	sudo nano /etc/systemd/system/uwsgi.service 
	
		[Unit]
		Description=uWSGI Emperor service
		[Service]
		ExecStartPre=/bin/bash -c 'mkdir -p /run/uwsgi; chown $USER:www-data /run/uwsgi'
		ExecStart=/usr/local/bin/uwsgi --emperor /etc/uwsgi/sites
		Restart=always
		KillSignal=SIGQUIT
		Type=notify
		NotifyAccess=all

		[Install]
		WantedBy=multi-user.target
	

 Note! replace $USER with username 

	start uwsgi service <br/>
	sudo service uwsgi start <br/>

## install nginx server
---------------------

	sudo apt update <br/>
	sudo apt install nginx <br/>
	sudo nano /etc/nginx/sites-available/firstsite <br/>
	
		server {
		    listen 80;
		    server_name localhost;

		    location = /favicon.ico { access_log off; log_not_found off; }
		    location /static/ {
			root /home/pc/firstsite;
		    }

		    location / {
			include         uwsgi_params;
			uwsgi_pass      unix:/run/uwsgi/firstsite.sock;
		    }
		}

	sudo ln -s /etc/nginx/sites-available/firstsite /etc/nginx/sites-enabled 
check configuration with <br/>
	
	sudo nginx -t 
restart nginx <br/>
	
	sudo systemctl restart nginx 
nginx firewall allow  <br/>
	
	sudo ufw allow 'Nginx Full' 
to start automatically at boot by typing: <br/>
	
	sudo systemctl enable nginx 
	sudo systemctl enable uwsgi 

delete the default page on /etc/nginx/sites-enable/default	<br/>	
	
	sudo rm /etc/nginx/sites-enabled/default 

type url for first django site using your ip address, in my case it's  <br/>
	
	http://35.224.171.124/ 
to access admin page use <br/>

	http://35.224.171.124/admin 
	username: admin 
	password: Pfix0insa 
	
## install docker on machine

	curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add - 
	sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

	sudo apt update <br/>

	sudo apt install -y docker-ce <br/>

	sudo usermod -aG docker ${user} <br/>
replace ${user} with your username <br/>
logout and login to affect the above cmd <br/>

## install rabbitmq using docker

	docker run -d --hostname ${hostname} --name my-rabit -p 8080:15672 rabbitmq 
replace ${hostname} with your hostname <br/>
	
	docker exec -it my-rabit /bin/bash 
to enable plugins type <br/>
	
	rabbitmq-plugins enable rabbitmq_management 
then type exit <br/>

now rabbitmq is running and working good u can test with the ff url: <br/>
	
	http://35.224.171.124:8080/ 
	username: guest 
	password:guest 

### install mysql on server

	sudo apt-get update 
	sudo apt-get install mysql-server 
	sudo systemctl enable mysql 

# PART 1.2 Create a simple task in django that will be run be run by celery

	 git clone https://github.com/demis-svenska/Django-Celery-Example.git 
	 cd Django-Celery-Example
	 sudo pip install django 
	 sudo pip install celery 
	 sudo pip install numpy 
	 sudo pip install scipy 
	 celery -A celery_try worker -l info 
	 python manage.py migrate 
	 python manage.py runserver 0.0.0.0:8090 
	 
 Then visit http://35.224.171.124:8090/index/. <br/>
 I have incountered an error on this while using rabbit docker container and executing the ff code.
 	
	celery -A celery_try worker -l info 
 
 It displays the following error
 
 	[2018-08-23 11:10:03,145: ERROR/MainProcess] consumer: Cannot connect to amqp://guest:**@127.0.0.1:5672//: [Errno 111] Connection refused. Trying again in 6.00 seconds...

it may take some time to fix it <br/>
 
 # PART 1.3 Running two apps on same server and using two apps on same broker
 
 ## Running 2 apps on same server
 
 ### using VirtualEnv and VirtualEnvWrapper
  
  install virtualenv, which can create Python virtual environments, and virtualenvwrapper, which adds some usability improvements to the virtualenv work flow.
  
	sudo -H pip install virtualenv virtualenvwrapper
	echo "export WORKON_HOME=~/Env" >> ~/.bashrc	
	echo "source /usr/local/bin/virtualenvwrapper.sh" >> ~/.bashrc	
	source ~/.bashrc	 
 to create virtual environment use mkvirtualenv cmd
 
 	mkvirtualenv secondsite
This will create a virtual environment, install Python and pip within it, and activate the environment.<br/>
i have already created first site on PART 1.1 <br/> our prompt will change to indicate that you are now operating within your new virtual environment. It will look something like this: (secondsite)user@hostname:~$  <br/>

to move out of the virtual environment use:

	deactivate cmd
to reactivate the virtual environment use:
	
	workon secondsite

 
 
 ## Using 2 apps on same broker
 
 ### using rabbitmq virtual host
 
 since am using rabbitmq docker machine, i need to login to container and execute the ff cmds.
 
 login to the rabbitmq docker container environment, that i created on part 1 <br/>
 
 	docker exec -it my-rabit /bin/bash
 
 Create virtual host for first site using cmd
 
 	rabbitmqctl add_vhost firstsite

create virtual host for second site using cmd

	rabbitmqctl add_vhost secondsite

U can now see the list of vhosts

	http://35.224.171.124:8080/#/vhosts

# PART 2 Scale


	

















