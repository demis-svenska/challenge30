steps
----------------install python,django,uwsgi,nginx on ubuntu 16.04 lTS---------------------

---------------install python, pip----------
sudo apt-get update
sudo apt-get install python-pip
sudo -H pip install --upgrade pip

---------------install virtualenv----------------------------
sudo -H pip install virtualenv virtualenvwrapper
echo "export WORKON_HOME=~/Env" >> ~/.bashrc
echo "source /usr/local/bin/virtualenvwrapper.sh" >> ~/.bashrc
source ~/.bashrc

----------install django------------
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

----------------install uwsgi-------------------------
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

start uwsgi service 
	sudo service uwsgi start


----------install nginx server-----------
sudo apt update
sudo apt install nginx
sudo nano /etc/nginx/sites-available/firstsite
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
check configuration with
	sudo nginx -t
restart nginx
	sudo systemctl restart nginx
nginx firewall allow 
	sudo ufw allow 'Nginx Full'
to start automatically at boot by typing:
	sudo systemctl enable nginx
	sudo systemctl enable uwsgi

delete the default page on /etc/nginx/sites-enable/default		
	sudo rm /etc/nginx/sites-enabled/default

type url for first django site using your ip address, in my case it's 
	http://35.224.171.124/

-----------install docker on machine---------------
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

sudo apt update

sudo apt install -y docker-ce

sudo usermod -aG docker ${user}
	replace ${user} with your username
logout and login to affect the above cmd

-----------install rabbitmq using docker------------
docker run -d --hostname ${hostname} --name my-rabit -p 8080:15672 rabbitmq
	replace ${hostname} with your hostname
docker exec -it my-rabit /bin/bash
   to enable plugins type
	rabbitmq-plugins enable rabbitmq_management
   then type exit

now rabbitmq is running and working good u can test with the ff url:
	http://35.224.171.124:8080/
	username: guest
	password:guest

-----------install mysql on server-----------------
sudo apt-get update
sudo apt-get install mysql-server
mysql_secure_installation
