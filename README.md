# resume-demo

how to deploy this project in vercel

## how to deploy on vps
```
git clone http ... .git
pip install -r requirements.txt



pip install gcc python3-devel uwsgi


uwsgi --http :8000 --module resume_demo.wsgi

sudo apt install nginx



sudo nano /etc/nginx/sites-available/resume-demo 

```





[Service]
User=root
Group=root
WorkingDirectory=/root/resume-demo
ExecStart=/root/virt/myenv/bin/gunicorn --workers 3 --bind unix:/root/resume-demo/resume-demo.sock resume_demo.wsgi:application

[Install]
WantedBy=multi-user.target






## How to deploy on vercel (it's huge, so it's not allowed on vercel, I failed)
1. allow vercel to access your github account
go to setting.py and change allowed host

=['.vercel.app', 'localhost', ']

2. create vercel.json in root

mistake: I missed comma here :

{
    "builds" : [{
        "src": "resume_demo.python",
        "use": "@vercel/python",
        "config": { "maxLambdaSize": "15mb", "runtime": "python3.9" }
    }],
    "routes": [
        {

3. go to wsgi.py and change this line
app = application


4. to deploy on vercel,  I delete pywin32 



deploy on render

choose postgresql 
install dj-dabase-url












How to package this project into Docker

1. create Dockerfile in root
put this project into this file , here my repo is called dockerhub, put it in dockerhub.
then create requirements.txt in dockerhub
input:
```

gunicorn==20.1.0
```


---
then create Dockerfile in dockerhub, input: 

```
FROM python:3.9.5-alpine

RUN pip install --upgrade pip

COPY ./requirements.txt .
RUN pip install -r requirements.txt

COPY ./resume_demo  /app

WORKDIR /app

COPY ./entrypoint.sh /
ENTRYPOINT ["sh", "/entrypoint.sh"]

```

Docker requires JSON array notation for the CMD and ENTRYPOINT instructions to use double quotes (") rather than single quotes ('). That's why you're seeing the error /bin/sh: [sh,: not found — it's trying to interpret [sh, as a command, which obviously doesn't exist.

Then create entrypoint.sh in dockerhub, input:

```
#!/bin/sh

python manage.py migrate --noinput
python manage.py collectstatic --noinput

gunicorn resume_demo.wsgi:application --bind 0.0.0.0:8000

```

Ensure that this script has execute permissions (chmod +x entrypoint.sh) so Docker can run it.
Then create docker-compose.yml in dockerhub, input:
In the nginx service, under the volumes section, you are missing a space after the hyphen - before static:/static.
```

version: '3.8'

services:
    django_gunicorn:
        volumes:
            - static:/app/staticfiles   # <-- change here
        env_file:
            - .env
        build:
            context: .
        ports:
            - "8000:8000"

    nginx:
        build: ./nginx
        volumes:
            - static:/app/staticfiles   # <-- and here
        ports:
            - "80:80"
        depends_on:
            - django_gunicorn

volumes:
    static:                            # <-- and here



```
Then go to setting.py and change SECRET_KEY to os.environ.get('SECRET_KEY')

```
SECRET_KEY = os.getenv('SECRET_KEY')

DEBUG = os.getenv('DEBUG') 


```





Then create .env in dockerhub, input:

```
SECRET_KEY=
DEBUG=FALSE
```

Then create nginx file in dockerhub, my static setting is:

STATICFILES_DIRS = [
    os.path.join(BASE_DIR, 'static'),
    os.path.join(BASE_DIR, 'mediafiles')
]

STATIC_URL = "/static/"
STATIC_ROOT = BASE_DIR / "staticfiles"

MEDIA_URL = "/mediafiles/"
MEDIA_ROOT = BASE_DIR / "mediafiles"




 create Dockerfile in nginx, then create default.conf in nginx, input in  default.conf:

```
upstream django {
    server django_gunicorn:8000;
}

server {
    listen 80;

    location / {
        proxy_pass http://django;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    location /static {
        alias /app/staticfiles/;
    }

    location /mediafiles {
        alias /app/mediafiles/;
    }
}

```

Then create Dockerfile in nginx, input:

```
FROM nginx:1.19.10-alpine

COPY ./default.conf /etc/nginx/conf.d/default.conf

```


Then in this repo, go to shell input:
    
    ```
    docker-compose up --build
    docker-compose up -d
    ```

use 
    
        ```

        docker-compose exec django_gunicorn python manage.py collectstatic

        docker exec -it dockerhub-django_gunicorn ls /app/staticfiles


        docker network ls

        docker volume ls
        docker volume inspect dockerhub_static

        sudo ls /var/lib/docker/volumes/dockerhub_static/_data
        sudo ls /var/lib/docker/volumes/dockerhub_static/_data/admin

        docker-compose down

        docker images

        docker rmi dockerhub_gunicorn
        docker rmi dockerhub_nginx

        docker volume prune # delete all volumes that are not associated with a container

        ```

        ```

check port
sudo ss -tuln | grep :80
sudo service nginx stop



asgiref==3.7.2
Django==4.2.6
django-ckeditor==6.7.0
django-js-asset==2.1.0
Pillow==9.5.0
pytz==2023.3.post1
sqlparse==0.4.4


how to delete docker volume
docker volume ls
 docker ps -a --filter "volume=dockerhub_static"
    docker stop 7c3c3c3c3c3c
    docker rm  7c3c3c3c3c3c

docker volume rm  dockerhub_static

docker exec -it dockerhub-nginx-1 /bin/sh
curl http://django_gunicorn:8000