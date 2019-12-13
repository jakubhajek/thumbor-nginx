# Image processing solution for creating image thumbnails

That application stacks allow you to easily setup a solution to upload images and create thumbnails on deamand. The only original images are stored. You can use one of cache solution from Thumbor or use e.g. Varnish to store thumbails.

The stack is designed to be deployed on Docker Swarm cluster.

Traefik is responsible for exposing services via Host Header and manaing SSL certiicates. Make sure to update stack files with the appropriate domains.

Here is a list of services which consist the entire stack.

## Thumbor

Thumbor - running as a service to create smart thumbnails. I added SECURITY KEY as environment variables but for test purposes you can remove that variables and use "unsafe" feature. See examples below. No additional configurate is required because I relay on built in image apsl/thumbor.

## Nginx for uploading images

Nginx - running as a service with Webdav enabled to upload images via POST.
See details in Nginx config file to learn more where the images will be stored and what IP addrsess are allowed to upload images. It will require changes according to your network settings.

You can easily upload images via

`curl -T '/path/to/image/image.jpeg http://nginx`

My goal was to only have access to uploading servies from local environment that's why this services is not exposed. It is only available from the local network that is `proxy-main`.

## Nginx for downloading thumbnails

Nginx - running as a service to get the original images and pass the url to Thumbor for creating thumbnail.
See nginx config for more details. The request is pass through to Thumbor for further processing.

## Traefik 2.x

Traefik working as a reverse proxy to each of the services.
There is a lot of changes in Traefik 2.x comparing to 1.7 so make sure you have added `labels` correctly in your services.

See Traefik stack file for more details and check the online documenation provided by Traefik team.

# Deployment

The first step is to deploy Traefik

`docker stack deploy -c stack-tr-main.yml traefik --prune`

The next step is deploying the stack with Thumbor and Nginx.

`docker stack deploy -c stack-image-processing.yml images --prune`

## Examples of usage

1. Upload image via curl command.
2. Download the image:

   If you disabled security you can use following example.
   https://get.replace-me.pl/unsafe/300x300/smart/orig.replace-me.pl/image.jpg

   If you enabled security which is stronlgy recommended use following url. Make sure you correctly generated /hash/ in URL. See Thumbor documentation for more details.

   https://get.replace-me.pl/HASH/300x300/smart/orig.replace-me.pl/image.jpg

## Docker Swarm

Docker Swarm as a platform to run, deploy and manange all running services.
