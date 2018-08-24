# cflat-server

## Build OS

- https://github.com/DieterReuter/image-builder-rpi64/releases

- flash --userdata ./user-data.yml ./hypriotos-rpi-v1.9.0.img


# This gives the [nextcloud] permissions to write to this directory since it runs as www-data
# - sudo mkdir -p /var/data/nextcloud
# - sudo setfacl -m u:www-data:rwx /var/data/nextcloud

https://jaxenter.de/docker-swarm-einfuehrung-65263

docker run --rm -it debian dpkg --print-architecture

traefik:
  image: traefik
  command: --web --docker --docker.watch --docker.domain=localhost --logLevel=DEBUG --entryPoints="Name:http Address::80"
  ports:
    - "80:80"
    - "8080:8080"
  volumes:
    - /var/run/docker.sock:/var/run/docker.sock
    - /dev/null:/traefik.toml

  # traefik
  - [
      docker, service, create,
        "--name" , "traefik",
        "--constraint=node.role==manager",
        "--publish", "80:80", 
        "--publish", "8080:8080",
        "--mount", "type=bind,source=/var/run/docker.sock,target=/var/run/docker.sock",
        "--network", "proxy",
        "arm64v8/traefik",
        "--docker",
        "--docker.swarmMode",
        "--docker.domain=traefik",
        "--docker.watch",
        "--api"
    ]
    
  # registry
  - [ sudo, mkdir, -p, /var/data/registry ]
  - [
      docker, service, create, 
       "--detach=false",
       "--name", "registry",
       "--publish", "5000:5000",
       "--replicas", "1",
       "--constraint", "node.role==manager",
       "--mount", "type=bind,src=/var/data/registry,dst=/var/lib/registry,ro",
       "--no-resolve-image",
       "cblomart/rpi-registry"
    ]

  # portainer
  - [ docker, volume, create, portainer_data ]
  - [ 
      docker, service, create, 
        "--detach=false", 
        "--name", "portainer", 
        "--publish", "9000:9000",
        "--replicas=1",
        "--constraint", "node.role == manager",
        "--mount", "type=bind,src=//var/run/docker.sock,dst=/var/run/docker.sock,ro", 
        "--mount", "type=volume,src=portainer_data,dst=/data", 
        "portainer/portainer", 
          "-H", "unix:///var/run/docker.sock", "--no-auth"
    ]



docker service create --detach=false --name registry --publish 5000:5000 --mount type=bind,src=/var/data/registry,dst=/var/lib/registry cblomart/rpi-registry:latest



docker run -d -p 80:80 -v nextcloud:/var/www/html -e SQLITE_DATABASE=cloud -e NEXTCLOUD_ADMIN_USER=admin -e NEXTCLOUD_ADMIN_PASSWORD=hypriot nextcloud:latest

     - "--api"
      - "--entrypoints=Name:http Address::80 Redirect.EntryPoint:https"
      - "--entrypoints=Name:https Address::443 TLS"
      - "--defaultentrypoints=http,https"
      - "--docker"
      - "--docker.swarmMode"
      - "--docker.domain=cflat-srv"
      - "--docker.watch"

# new

  # shepherd
  - [
      docker, service, create, 
       "--detach=false", 
       "--name", "shepherd",
       "--replicas", "1",
       "--constraint", "node.role==manager",
       "--env", "SLEEP_TIME='1440m'",
       "--mount", "type=bind,src=//var/run/docker.sock,dst=/var/run/docker.sock,ro", 
       "mazzolino/shepherd"
    ]


curl -i http://localhost:5000/v2/

rm -rf cflat-server && curl -L https://github.com/cgHome/cflat-server/tarball/master | tar xz --strip=1 --one-top-level=cflat-server

docker-compose -f docker-compose.yml bundle --push-images
docker-compose -f docker-compose.yml --verbose bundle --push-images
docker stack deploy --bundle-file stack.dab 


docker stack deploy --bundle-file $(docker-compose -f docker-compose.yml bundle --push-images)


docker service create \
    --name shepherd \
    --no-resolve-image \
    --replicas 1 \
    --constraint "node.role==manager" \
    --env SLEEP_TIME="5m" \
    --env BLACKLIST_SERVICES="shepherd" \
    --mount type=bind,source=/var/run/docker.sock,target=/var/run/docker.sock,ro \
    mazzolino/shepherd

docker run \
  --name=cadvisor \
  --detach=true \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:rw \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --publish=88:8080 \
  --privileged \
  --device-cgroup-rule 'c 195:* mrw' \
  braingamer/cadvisor-arm:latest
