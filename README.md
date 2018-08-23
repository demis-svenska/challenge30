

# PART 1.1
# install python,django,celery,uwsgi,nginx on ubuntu 16.04 LTS


## install python, pip
-------------------------
sudo apt-get update <br/>
sudo apt-get install python-pip <br/>
sudo -H pip install --upgrade pip <br/>

## install virtualenv
-------------------------------------------
sudo -H pip install virtualenv virtualenvwrapper <br/>
echo "export WORKON_HOME=~/Env" >> ~/.bashrc <br/>
echo "source /usr/local/bin/virtualenvwrapper.sh" >> ~/.bashrc <br/>
source ~/.bashrc

## install django and create first site
----------------------
mkvirtualenv firstsite  <br/>
pip install django <br/>
pip install celery  <br/>
cd ~ <br/>
django-admin.py startproject firstsite <br/>
cd ~/firstsite <br/>
~/firstsite/manage.py migrate <br/>
~/firstsite/manage.py createsuperuser <br/>
	username: admin <br/>
	password:Pfix0insa <br/>
nano ~/firstsite/firstsite/settings.py <br/>
	ALLOWED_HOSTS = ['your-ip-address', 'localhost'] and <br/>
	add STATIC_ROOT = os.path.join(BASE_DIR, 'static/') <br/>
~/firstsite/manage.py collectstatic <br/>
test running:
	~/firstsite/manage.py runserver 0.0.0.0:8080

## install uwsgi
-----------------------------------------
sudo -H pip install uwsgi	<br/>
sudo mkdir -p /etc/uwsgi/sites <br/>
sudo nano /etc/uwsgi/sites/firstsite.ini <br/>
	
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

 Note! replace $USER with username  <br/><br/>

sudo nano /etc/systemd/system/uwsgi.service <br/>
	
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
	
 <br/>
 Note! replace $USER with username <br/>

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
sudo ln -s /etc/nginx/sites-available/firstsite /etc/nginx/sites-enabled <br/>
check configuration with <br/>
	sudo nginx -t <br/>
restart nginx <br/>
	sudo systemctl restart nginx <br/>
nginx firewall allow  <br/>
	sudo ufw allow 'Nginx Full' <br/>
to start automatically at boot by typing: <br/>
	sudo systemctl enable nginx <br/>
	sudo systemctl enable uwsgi <br/>

delete the default page on /etc/nginx/sites-enable/default	<br/>	
	sudo rm /etc/nginx/sites-enabled/default <br/>

type url for first django site using your ip address, in my case it's  <br/>
	http://35.224.171.124/ <br/>
to access admin page use <br/>
http://35.224.171.124/admin <br/>
username: admin <br/>
password: Pfix0insa <br/>
## install docker on machine

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add - <br/>

sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"<br/>

sudo apt update <br/>

sudo apt install -y docker-ce <br/>

sudo usermod -aG docker ${user} <br/>
	replace ${user} with your username <br/>
logout and login to affect the above cmd <br/>

## install rabbitmq using docker

docker run -d --hostname ${hostname} --name my-rabit -p 8080:15672 rabbitmq <br/>
	replace ${hostname} with your hostname <br/>
docker exec -it my-rabit /bin/bash <br/>
   to enable plugins type <br/>
	rabbitmq-plugins enable rabbitmq_management <br/>
   then type exit <br/>

now rabbitmq is running and working good u can test with the ff url: <br/>
	http://35.224.171.124:8080/ <br/>
	username: guest <br/>
	password:guest <br/>

### install mysql on server

sudo apt-get update <br/>
sudo apt-get install mysql-server <br/>
sudo systemctl enable mysql <br/>

# PART 1.2 Create a simple task in django that will be run be run by celery


 git clone https://github.com/demis-svenska/Django-Celery-Example.git <br/>
 cd Django-Celery-Example <br/>
 sudo pip install django <br/>
 sudo pip install celery <br/>
 sudo pip install numpy <br/>
 sudo pip install scipy <br/>
 celery -A celery_try worker -l info <br/>
 python manage.py migrate <br/>
 python manage.py runserver 0.0.0.0:8090 <br/>
 Then visit http://35.224.171.124:8090/index/. <br/>

















