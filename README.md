# Documentaion
Prerequisites:

# update your local packages 
1.sudo apt-get update
# install dependencies
2. sudo apt-get install python3-pip python3-dev nginx

python3 -m venv <your venv>
  
source yourvitualenv/bin/activate

pip3 install gunicorn flask

# Create a file named app.py


from flask import Flask
app = Flask(__name__)
@app.route('/')
def hello_world():
    return "Hello World!"
if __name__ == '__main__':
  app.run(debug=True,host='0.0.0.0')
  
python app.py
 
# Create the WSGI Entry Point

from app import app

if __name__ == "__main__":
    app.run()
 
# Testing Gunicorn’s Ability to Serve the Project

cd ~/src
gunicorn --bind 0.0.0.0:5000 wsgi:app


# Now deactivate virtualenv by following command:

deactivate


# systemd unit file will allow Ubuntu’s init system to automatically start Gunicorn and serve our Flask application whenever the server boots.
# Create a unit file ending in .service within the /etc/systemd/system directory to begin :

sudo vim /etc/systemd/system/app.service


[Unit]
#  specifies metadata and dependencies
Description=Gunicorn instance to serve myproject
After=network.target
# tells the init system to only start this after the networking target has been reached
# We will give our regular user account ownership of the process since it owns all of the relevant files
[Service]
# Service specify the user and group under which our process will run.
User=yourusername
# give group ownership to the www-data group so that Nginx can communicate easily with the Gunicorn processes.
Group=www-data
# We'll then map out the working directory and set the PATH environmental variable so that the init system knows where our the executables for the process are located (within our virtual environment).
WorkingDirectory=/home/tasnuva/work/deployment/src
Environment="PATH=/home/tasnuva/work/deployment/src/myprojectvenv/bin"
# We'll then specify the commanded to start the service
ExecStart=/home/tasnuva/work/deployment/src/myprojectvenv/bin/gunicorn --workers 3 --bind unix:app.sock -m 007 wsgi:app
# This will tell systemd what to link this service to if we enable it to start at boot. We want this service to start when the regular multi-user system is up and running:
[Install]
WantedBy=multi-user.target





sudo systemctl start app
sudo systemctl enable app


# Configuring Nginx



# We’ll need to tell NGINX about our app and how to serve it.

sudo nano /etc/nginx/sites-available/app


server {
    listen 80;
    server_name server_domain_or_IP;

location / {
  include proxy_params;
  proxy_pass http://unix:/home/tasnuva/work/deployment/src/app.sock;
    }
}


# Enable Nginx server block

sudo ln -s /etc/nginx/sites-available/app /etc/nginx/sites-enabled

sudo systemctl restart nginx

# http://server_domain_or_IP




