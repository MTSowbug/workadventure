![](https://github.com/thecodingmachine/workadventure/workflows/Continuous%20Integration/badge.svg) [![Discord](https://img.shields.io/discord/821338762134290432?label=Discord)](https://discord.gg/G6Xh9ZM9aR) ![Awesome](https://awesome.re/badge.svg)

![WorkAdventure office image](README-MAP.png)

# WorkAdventure


WorkAdventure is a platform that allows you to design **fully customizable collaborative virtual worlds** (metaverse). 

With your own avatar, you can **interact spontaneously** with your colleagues, clients, partners (using a **video-chat system**, triggered when you approach someone).
Imagine **all types of immersive experiences** (recruitments, onboarding, trainings, digital workplace, internal/external events) on desktop, mobile or tablet.

_The little plus? The platform is **GDPR** and **open source**!_

**See more features for your [virtual office](https://workadventu.re/virtual-offices/virtual-meetings/?utm_source=github)!**

**Pricing for our [SaaS version](https://workadventu.re/pricing/?utm_source=github)!**


[![Workadventure live demo example](https://workadventu.re/wp-content/uploads/2024/02/Button-Live-Demo.png)](https://play.staging.workadventu.re/@/tcm/workadventure/wa-village/?utm_source=github)
[![Workadventure Website](https://workadventu.re/wp-content/uploads/2024/02/Button-Website.png)](https://workadventu.re/?utm_source=github)


###### Support our team!
[![Discord Logo](https://workadventu.re/wp-content/uploads/2024/02/Icon-Discord.png)](https://discord.com/invite/G6Xh9ZM9aR)
[![X Social Logo](https://workadventu.re/wp-content/uploads/2024/02/Icon-X.png)](https://twitter.com/Workadventure_)
[![LinkedIn Logo](https://workadventu.re/wp-content/uploads/2024/02/Icon-LinkedIn.png)](https://www.linkedin.com/company/workadventu-re/)


![Stats repo](https://github-readme-stats.vercel.app/api?username={username}&theme=transparent)



## Community resources

1. Want to build your own map, check out our **[map building documentation](https://docs.workadventu.re/map-building/)**
2. Check out resources developed by the WorkAdventure community at **[awesome-workadventure](https://github.com/workadventure/awesome-workadventure)**

## Setting up a production environment

We support 2 ways to set up a production environment:

- using Docker Compose
- or using a Helm chart for Kubernetes

Please check the [Setting up a production environment](docs/others/self-hosting/install.md) guide for more information.

> [!NOTE]
> WorkAdventure also provides a [hosted version](https://workadventu.re/?utm_source=github) of the application. Using the hosted version is
> the easiest way to get started and helps us to keep the project alive.

### AWS EC2 quickstart (Docker Compose)

The steps below summarize how to bootstrap a production-grade WorkAdventure instance on an Ubuntu-based AWS EC2
instance using Docker Compose, alongside the more detailed [self-hosting documentation](docs/others/self-hosting/install.md)
and [Docker deployment guide](contrib/docker/README.md).

1. **Prepare your AWS infrastructure**
   - Allocate a domain name (for example, `wa.example.com`) and create an A record in Route 53 or your DNS provider that points to the EC2 instance's Elastic IP. WorkAdventure issues HTTPS certificates automatically via Let's Encrypt, so the hostname must be reachable on ports 80 and 443.【F:contrib/docker/README.md†L12-L24】【F:contrib/docker/README.md†L42-L53】
   - Plan additional DNS records for your Jitsi and Coturn servers if you self-host real-time services, as WorkAdventure expects them to live on their own hostnames.【F:docs/others/self-hosting/install.md†L31-L57】
   - Create (or update) the instance security group to allow inbound TCP 22 (SSH), 80 (HTTP), and 443 (HTTPS). Open the ports required by your Coturn/Jitsi stack as well (for example, UDP/TCP 3478 for Coturn).

2. **Launch the EC2 host**
   - Start an Ubuntu 22.04 LTS (Jammy) instance with at least 2 vCPUs and 4 GiB RAM (for example, the `t3a.medium` family). This sizing aligns with the reference architecture for roughly 300 concurrent users; adjust according to your usage and video infrastructure.【F:contrib/docker/README.md†L16-L23】
   - Attach an Elastic IP so your DNS record stays stable across reboots.

3. **Install Docker Engine, the Compose plugin, and supporting tools**
   ```bash
   sudo apt-get update
   sudo apt-get install -y ca-certificates curl gnupg git
   sudo install -m 0755 -d /etc/apt/keyrings
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
   echo "deb [arch=\$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
     \$(. /etc/os-release && echo \$VERSION_CODENAME) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   sudo apt-get update
   sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
   sudo systemctl enable docker
   sudo usermod -aG docker $USER
   newgrp docker
   ```
   - Reconnect your SSH session if you skip the `newgrp` command so the Docker group membership applies.

4. **Download the deployment files**
   ```bash
   git clone https://github.com/thecodingmachine/workadventure.git
   cd workadventure/contrib/docker
   cp .env.prod.template .env
   cp docker-compose.prod.yaml docker-compose.yaml
   ```
   Keeping the deployment in `/opt/workadventure` or another dedicated directory helps with backups and upgrades.

5. **Configure `.env` for production**
   - Generate a long random secret and set `SECRET_KEY`, for example: `openssl rand -hex 32`.
   - Set `DOMAIN` to the hostname you mapped in DNS and provide an email in `ACME_EMAIL` so Let's Encrypt can issue certificate expiry notices.【F:contrib/docker/README.md†L42-L53】【F:contrib/docker/.env.prod.template†L15-L20】【F:contrib/docker/.env.prod.template†L98-L104】
   - Choose a released WorkAdventure version (for example, `v1.15.3`) and assign it to `VERSION` so your compose file stays in sync with the container images.【F:contrib/docker/README.md†L55-L79】
   - Define credentials for the map storage service (`MAP_STORAGE_AUTHENTICATION_USER` and `MAP_STORAGE_AUTHENTICATION_PASSWORD`) and any other options relevant to your setup (Jitsi, Coturn, OpenID, Matrix, etc.).
   - Review the rest of the variables in `.env` and update them to reflect your infrastructure choices, especially if you run external Jitsi, Coturn, or Matrix services.【F:contrib/docker/.env.prod.template†L34-L152】

6. **Start WorkAdventure**
   ```bash
   docker compose up -d
   docker compose ps
   docker compose logs -f
   ```
   - Confirm every container is healthy before proceeding. Traefik will automatically request and renew TLS certificates for the configured domain.【F:contrib/docker/README.md†L80-L109】【F:contrib/docker/README.md†L200-L210】

7. **Upload your first map**
   - Build or download a map using the [map starter kit](https://github.com/workadventure/map-starter-kit), then follow the [map upload guide](https://docs.workadventu.re/map-building/tiled-editor/publish/wa-hosted) to publish it.
   - Visit `https://<your-domain>/map-storage/` in a browser, authenticate with the credentials you configured, and verify your map is listed.【F:contrib/docker/README.md†L110-L135】

8. **Plan your next steps**
   - Configure Jitsi and Coturn so larger groups and restrictive networks can join reliably.【F:contrib/docker/README.md†L136-L158】
   - Integrate authentication (OpenID Connect) and optional Matrix chat support as needed.【F:contrib/docker/README.md†L160-L192】
   - Subscribe to security alerts by setting `SECURITY_EMAIL` in `.env` and monitor container updates regularly.

By completing the steps above you will have a functional EC2-hosted WorkAdventure instance that can be upgraded by replacing the compose files with newer release versions and re-running `docker compose up -d --force-recreate` during maintenance windows.【F:contrib/docker/README.md†L172-L195】

## Setting up a development environment

> [!NOTE]
> These installation instructions are for local development only. They will not work on
> remote servers as local environments do not have HTTPS certificates.

Install Docker and clone this repository.

> [!WARNING]
> If you are using Windows, make sure the End-Of-Line character is not modified by the cloning process by setting
> the `core.autocrlf` setting to false: `git config --global core.autocrlf false`

Run:

```
cp .env.template .env
docker-compose up
```

The environment will start with the OIDC mock server enabled by default.

You should now be able to browse to http://play.workadventure.localhost/ and see the application.
You can view the Traefik dashboard at http://traefik.workadventure.localhost

(Test user is "User1" and password is "pwd")

If you want to disable the OIDC mock server (for anonymous access), you can run:

```console
$ docker-compose -f docker-compose.yaml -f docker-compose-no-oidc.yaml up
```

Note: on some OSes, you will need to add this line to your `/etc/hosts` file:

**/etc/hosts**
```
127.0.0.1 oidc.workadventure.localhost redis.workadventure.localhost play.workadventure.localhost traefik.workadventure.localhost matrix.workadventure.localhost extra.workadventure.localhost icon.workadventure.localhost map-storage.workadventure.localhost uploader.workadventure.localhost maps.workadventure.localhost api.workadventure.localhost front.workadventure.localhost
```


### Troubleshooting

See our [troubleshooting guide](docs/others/troubleshooting.md). 
