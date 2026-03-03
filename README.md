# Minimal config for running VaultWarden NGINX and Fail2Ban behind a Cloudflare tunnel

### This configuraion also uses Let's Encrypt certificates for local network hosts

Docker needs to be setup for syslog so that fail2ban can monitor the access log. 
Use the following for /etc/docker/daemon.json

```
{
  "log-driver": "syslog",
  "log-opts": {
    "syslog-address": "unixgram:///dev/log",
    "tag": "docker/{{.Name}}"
  }
}
```

The nginx.conf file needs to be modified to suit your needs. Specifically:

Change `<your_net/cidr>` to whatever your local network is (i.e. `192.168.0.0/16`, `192.168.1.0/24`,`172.16.0.0/12`, ...).

Add the name of your server in the server{} blocks.

`<full_server_name>` -> `host.domain.com`

Be sure to change this on the ssl_certificate lines as well.

You will also need a .env file that contains your cloudflare tunnel token formatted as:

`export TUNNEL_TOKEN="<token_id>"`

If you want to run the VaultWarden admin interface, you will also need to add the admin token to the .env file

`export VAULTWARDEN_ADMIN_TOKEN='<admin_token>'`

The api calls to update the cloudflare block list were **especially** difficult to figure out. 
Many tutorials point to paid versions of cloudflare block list methods. 
This configuration works with the free tier.

You will need to create a block list on Cloudflare for Fail2Ban to update. Go to

`Dashboard -> Configurations -> Lists Tab`

and create a list with IP as the type.

Then, on the domain your tunnel is running, go to:

`Security -> Security Rules`

and add a Custom Rule where:
`Field` = `IP Source Address`
`Operator` = `is in list`
`Value` = Your List Name

Ensure `Action` is `Block`


This should be a barebones config for this to get up and running.

Certificate updating is performed by the cerbot container
Running the following command *Should* update the cert:
	`docker compose run --rm certbot certonly --webroot --webroot-path /var/www/certbot/ -d <domain_you_own>`

I used this website to get the letsencrypt portion working:
	https://phoenixnap.com/kb/letsencrypt-docker

