Docker image for [Timeoff](http://timeoff.management/) tool
-----------------------------------------------------------

# Using the docker image

## Testing on local

1. Pull the docker image from [Docker Hub](https://hub.docker.com/r/thperret/timeoff-management/):
`docker pull thperret/timeoff-management`

2. Create and run the container
`docker run thperret/timeoff-management`

3. Open your browser to `http://localhost:3000`

## Use in production

By default, the timeoff container will use a sqlite database and a no smtp server will be configured.

To configure a production database, set `NODE_ENV` environment variable to `production` and set corresponding MYSQL variables:

`docker run -e NODE_ENV=production -e MYSQL_HOST=mysql-host.example.org -e MYSQL_USER=timeoff -e MYSQL_DATABASE=timeoff -e MYSQL_PASSWORD=timeoff thperret/timeoff-management`

You can also configure the container using `docker-compose`. See [examples](https://github.com/thperret/timeoff-management_docker/blob/master/examples)

## Environment variables

You can set the following variables to configure the `timeoff` container:

 Variable name | Configuration | Default | Possible values | Remarks
---------------|---------------|---------|-----------------|---------
`NODE_ENV` | Set environment | `development` | `development`, `production`, `test` | You should always use `production`
`SENDER_MAIL` | Mails from | None | email address | Needed for enabling mail sending
`SMTP_HOST` | smtp server host | None | host | Needed for enabling mail sending
`SMTP_PORT` | smtp server port | None | port | Needed for enabling mail sending
`SMTP_USER` | smtp username | None | username/address | Needed for enabling mail sending
`SMTP_PASSWORD` | smtp password | None | password | Needed for enabling mail sending
`APP_URL` | Set application URL in sent mails | `http://app.timeoff.management` | URL | You should set this
`PROMOTION_URL` | Set url in footer mails | `http://timeoff.management` | URL | You can change this if you want footer mail link to redirect to your hosted application
`ALLOW_ACCOUNTS_CREATION` | Enable/Disable public companies account creation | `true` | `true` , `false` | You need to enable account creation at least on first run to create your company. You can disable it afterwards and restart the container

# Building the docker image

1. Clone the repo:
`git clone https://github.com/thperret/timeoff-management_docker.git`

2. Change working directory:
`cd timeoff-management_docker`

3. Build image:
`docker build -t timeoff-management:latest .`

# Running in local environment

I tried using docker-compose.yaml file in examples/docker-compose.yml, unsuccesfully.  It had to do mainly with two factors:

a) It uses a mariadb image, which can't use volumes.  
b) The database takes longer to start, so the timeoff app container throws an error.  

The way I remedied it was by using a mysql image instead of the mariadb one.  However, I ran into the same problem with the db taking too long to start. So then I came up with the following commands from that docker-compose example file:

`docker run --name timeoff_db -p3308:3306 -e MYSQL_ROOT_PASSWORD=timeoff -e MYSQL_DATABASE=timeoff -e MYSQL_USER=timeoff -e MYSQL_PASSWORD=timeoff mysql:5.6.45`

I waited for the DB to be capable of accepting connections, about 10 seconds.

`docker run --name timeoff_app -p3001:3000 --link timeoff_db -e ALLOW_ACCOUNTS_CREATION=true -e NODE_ENV=production -e MYSQL_HOST=timeoff_db -e MYSQL_USER=timeoff -e MYSQL_DATABASE=timeoff -e MYSQL_PASSWORD=timeoff thperret/timeoff-management:0.6.2`
