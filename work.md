# cflat-server

## Build OS

- flash --userdata ./user-data.yml ./hypriotos-rpi-v1.9.0.img


# This gives the [nextcloud] permissions to write to this directory since it runs as www-data
# - sudo mkdir -p /var/data/nextcloud
# - sudo setfacl -m u:www-data:rwx /var/data/nextcloud

https://jaxenter.de/docker-swarm-einfuehrung-65263

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
        "--name", "traefik",
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


docker run -d -p 80:80 -v nextcloud:/var/www/html -e SQLITE_DATABASE=cloud -e NEXTCLOUD_ADMIN_USER=admin -e NEXTCLOUD_ADMIN_PASSWORD=hypriot nextcloud:latest

     - "--api"
      - "--entrypoints=Name:http Address::80 Redirect.EntryPoint:https"
      - "--entrypoints=Name:https Address::443 TLS"
      - "--defaultentrypoints=http,https"
      - "--docker"
      - "--docker.swarmMode"
      - "--docker.domain=cflat-srv"
      - "--docker.watch"