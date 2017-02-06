# GitLab

SystemDocker units for [GitLab](https://about.gitlab.com/) CE, EE and Runner.

GitLab EE dropin coming soon. Subscribe to [SystemDocker/gitlab#2](https://github.com/SystemDocker/gitlab/issues/2) for updates on this.

# GitLab Installation Notes

Install the `gitlab@.service` as you would normally for any other [SystemDocker](https://systemdocker.github.io/) service, adding the `systemdocker.conf` dropin from [SystemDocker Common](https://github.com/SystemDocker/common). It is assumed that [SystemDocker Common](https://github.com/SystemDocker/common) has been downloaded and installed before attempting to install this GitLab service.

```
git clone https://github.com/SystemDocker/gitlab.git /srv/docker/gitlab
cd /etc/systemd/system
ln -s /srv/docker/gitlab/units/gitlab\@.service
mkdir -p gitlab\@.service.d
cd gitlab\@.service.d
ln -s /srv/docker/common/dropins/systemdocker.conf 00-systemdocker.conf
```

Use the `units/gitlab@.service.d/templates/secrets.conf` template to create a secrets droping making sure to generate your own secrets before use. You should generate new secrets for each instance of GitLab that you run. For an instance named `example` you would run the following:

```
cd /etc/systemd/system
mkdir -p gitlab\@example.service.d
cd gitlab\@example.service.d
cp /srv/docker/gitlab/units/gitlab\@.service.d/templates/secrets.conf 10-secrets.conf
# Edit `secrets.conf` with your favourite editor to change the secrets. Secrets can be generated with `pwgen -Bsv1 64`
```

GitLab also requires a database and cache to be setup along-side the service for it work. It is recomended that the [SystemDocker PostgreSQL](https://github.com/SystemDocker/postgresql) and [SystemDocker Redis](https://github.com/SystemDocker/redis) services are used. Each provide dropins (`dropin/postgresql-link.conf` and `dropin/redis-link.conf` respectively) which can be installed as part of the service by linking as a dropin for `gitlab@.service`:

```
cd /etc/systemd/system/gitlab\@.service.d
ln -s /srv/docker/postgresql/dropins/postgresql-link.conf 10-postgresql-link.conf
cp /srv/docker/redis/dropins/redis-link.conf 10-redis-link.conf
sed -i "s/:redis'\$/:redisio'/" 10-redis-link.conf
```

**NOTE:** Currently the image used for GitLab expects redis to be linked as `redisio` instead of `redis`, this is why `redis-link.conf` is copied and modified. Subscribe to [SystemDocker/gitlab#1](https://github.com/SystemDocker/gitlab/issues/1) for updates on this issue.

You should ensure that [SystemDocker PostgreSQL](https://github.com/SystemDocker/postgresql) and [SystemDocker Redis](https://github.com/SystemDocker/redis) have been installed before trying to start the `gitlab` service.

See the [sameersbn/gitlab](https://github.com/sameersbn/docker-gitlab) Docker image documentation for more details of the image configuration options.
