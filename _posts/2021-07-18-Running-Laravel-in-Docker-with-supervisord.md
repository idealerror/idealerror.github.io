---
title: Running Laravel in Docker with supervisord
author: Chris Geiger
date: 2021-07-18 12:00:00 -0700
categories: [Programming,DevOps]
tags: [laravel,docker,programming]
math: true
mermaid: true
image:
  src: https://www.chrisgeiger.dev/assets/img/posts/supervisord_article.jpg
  width: 850
  height: 400
---

Ask any DevOps or Systems Engineer and they'll agree that a Docker container should only run a single process.  This is because of the single process dies, the container should die with it.  What if you need to run multiple processes on a single container?  Well that's where supervisord comes in.  

# What is supervisord?
Supervisord is a linux application/service that allows you to run multiple applications as background processes (or foreground processes) utilizing a single service.  With this, supervisord will also monitor and restart processes if they fail.  Supervisord also has the ability to control 
- When an application starts
- Where an application logs to
- What user the application runs as

# Why use supervisord with Laravel?
Laravel has multiple processes that utilize the same codebase such as the scheduler and worker.  Supervisord makes it easy to run the web server, scheduler, and queue worker all on the same machine.  If for some reason your application processes heavy loads, it may make sense to split these into multiple containers.  For most simple applications though (or dev), you can just combine them into a single container.

# How to implement supervisord on Docker
## Dockerfile
It's fairly simple, you just need to make sure that you install supervisord and start it at the end of the Dockerfile. 

Here's an example utilizing the image `php:7.4-apache`

```dockerfile
FROM php:7.4-apache

# Install supervisord
RUN apt-get update && apt-get upgrade -y && apt-get install -y supervisor

# Make supervisor log directory
RUN mkdir -p /var/log/supervisor

# Copy local supervisord.conf to the conf.d directory
COPY --chown=root:root supervisord.conf /etc/supervisor/conf.d/supervisord.conf

# Start supervisord
`CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/supervisord.conf"]`
```

## supervisord.conf 
As mentioned above, the supervisord.conf file contains all of the applications that will be running on the container.  Below you will see that I've separated it out into 3 processes: worker, apache2, and schedule.  Apache starts first with the priority 1, then worker, then schedule.  Schedule is actually using a bash script instead of cron because cron uses it's own environment variables. 

```bash
[supervisord]
nodaemon = true
logfile = /dev/null
logfile_maxbytes = 0
pidfile = /run/supervisord.pid

[program:worker]
directory=/var/www/html
process_name=%(program_name)s_%(process_num)02d
command=php artisan queue:work --sleep=3 --tries=3
autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
user=root
numprocs=2
redirect_stderr=true
stdout_logfile = /dev/fd/1
stdout_logfile_maxbytes=0
stderr_logfile = /dev/fd/2
stderr_logfile_maxbytes=0
redirect_stderr=true
stopwaitsecs=3600
priority = 6

[program:apache2]
command=/usr/sbin/apache2ctl -DFOREGROUND
killasgroup=true
stopasgroup=true
stdout_logfile = /dev/fd/1
stdout_logfile_maxbytes=0
stderr_logfile = /dev/fd/2
stderr_logfile_maxbytes=0
redirect_stderr=true
autostart=true
autorestart=true
user=root
priority = 1

[program:schedule]
command = /bin/bash -c "/var/www/html/schedule.sh"
stdout_logfile = /dev/fd/1
stdout_logfile_maxbytes=0
stderr_logfile = /dev/fd/2
stderr_logfile_maxbytes=0
user = root
autostart = true
autorestart = true
priority = 20
```

## schedule.sh
The only major problem with using this method to run cron jobs is it doesn't happen at the top of the minute, it happens every 60 seconds from when the script is executed.  I'm sure this can be updated to start at the top of the minute, it just doesn't matter to me much.
```bash
while [ true ]
do
 php /var/www/html/artisan schedule:run --verbose --no-interaction &
 sleep 60
done
```

