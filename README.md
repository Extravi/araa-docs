# tailsx-docs

[![Counter](https://visitor-badge.laobi.icu/badge?page_id=Extravi.tailsxdocs)](https://github.com/Extravi/tailsx-docs)
## Table of contents
- [tailsx-docs](#tailsx-docs)
  - [Table of contents](#table-of-contents)
- [Basic steps](#basic-steps)
  - [nginx](#nginx)
  - [SSL/TLS certificate](#ssltls-certificate)
  - [Firewall (Allow only HTTPS/HTTP/SSH)](#firewall-allow-only-httpshttpssh)
  - [Disable IPv6](#disable-ipv6)
  - [Final setup \& starting Tailsx](#final-setup--starting-tailsx)
- [Manually start TailsX](#manually-start-tailsx)
  - [opensearch.xml](#opensearchxml)
    - [Generate the opensearch](#generate-the-opensearch)
    - [Manual configuration](#manual-configuration)
  - [Install PIP packages \& start the server](#install-pip-packages--start-the-server)
- [Test your server](#test-your-server)

# Basic steps

To write this guide, I was using Ubuntu Server 22.04 LTS. This guide assumes you're using sudo or root.

Install required packages:
```bash
apt install nginx nginx-extras certbot python3-certbot-nginx
```

## nginx

Configure nginx by replacing `tailsx.yourdomain.com` with your own domain:
```bash
cd /etc/nginx/sites-enabled/
rm default
wget -O tailsx https://raw.githubusercontent.com/Extravi/tailsx-docs/main/config/tailsx
nano tailsx
```

Now cd into /etc/nginx/ and replace nginx.conf; this will disable logging and improve server security:
```bash
cd /etc/nginx/
rm nginx.conf
wget -O nginx.conf https://raw.githubusercontent.com/Extravi/tailsx-docs/main/config/nginx.conf
nginx -t && nginx -s reload
```

Expected output:
```bash
root@ubuntu-s-1vcpu-1gb-tor1-01:/etc/nginx# nginx -t && nginx -s reload
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
root@ubuntu-s-1vcpu-1gb-tor1-01:/etc/nginx#
```

## SSL/TLS certificate
Obtain an SSL/TLS certificate, but before doing so, make sure you have an A record pointed to your server's IPv4 address:
```bash
certbot --nginx -d tailsx.yourdomain.com
```

Once you've done that, open the crontab file:
```bash
crontab -e
```

Then paste this at the bottom of the crontab file. This will automatically renew your Let’s Encrypt certificate:
```bash
0 12 * * * /usr/bin/certbot renew --quiet
```

## Firewall (Allow only HTTPS/HTTP/SSH)
Setup a firewall with UFW:
```bash
ufw default deny
ufw allow ssh
ufw allow https
ufw allow http
ufw enable
```

Run the status command:
```bash
ufw status verbose
```

You should see an output like this:
```bash
root@ubuntu-s-1vcpu-1gb-tor1-01:~/tailsx# ufw status verbose
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere
443                        ALLOW IN    Anywhere
80/tcp                     ALLOW IN    Anywhere
22/tcp (v6)                ALLOW IN    Anywhere (v6)
443 (v6)                   ALLOW IN    Anywhere (v6)
80/tcp (v6)                ALLOW IN    Anywhere (v6)

root@ubuntu-s-1vcpu-1gb-tor1-01:~/tailsx#
```

## Disable IPv6
Now we need to disable IPv6 because many websites, like Google, are more likely to block IPv6:
```bash
bash -c 'cat <<EOF >> /etc/sysctl.conf
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
EOF'
```

Now you need to configure your SSH daemon to only listen over IPv4:
```bash
echo 'AddressFamily inet' | sudo tee -a /etc/ssh/sshd_config
```

## Final setup & starting Tailsx
Clone TailsX and cd into the directory:
```bash
git clone https://github.com/Extravi/tailsx.git
cd tailsx/
```

You can chose to setup & start Tailsx in one of two ways;
- [Manually](/README.md#manually-start-tailsx)
- [With Docker](/Docker.md)

# Manually start TailsX
Install some more required apt packages;
```bash
apt install python3-venv python3-pip python3
```

## opensearch.xml
Configure the server's `opensearch.xml`. You can choose 1 of the following ways to do it;

### Generate the opensearch
The shell script `/scripts/generate-opensearch.sh` can automatically generate the server's `opensearch.xml` file for you. All you need to do is export a DOMAIN environmental variable before running the script;
```bash
cd tailsx/
export DOMAIN=https://your.domain.com #Modify this variable to be your domain. Note the use of https://!
sh scripts/generate-opensearch.sh
```

### Manual configuration
Configure `opensearch.xml` by replacing `http://127.0.0.1:5000/` with `https://your.domain.com/`, making sure to replace `http://` with `https://`:
```bash
cd tailsx/
cd static/
mv opensearch.xml.example opensearch.xml
nano opensearch.xml
cd ..
```

## Install PIP packages & start the server
Setup a Python virtual environmnent & activate it:
```bash
python3 -m venv venv/
. venv/bin/activate
```

Install the required PIP packages:
```bash
pip install -r requirements.txt
```

Finally, run this command to start TailsX:
```bash
gunicorn -w 4 __init__:app
```
<i>Read the gunicorn documentation to understand more about how the server works; [Running Gunicorn](https://docs.gunicorn.org/en/latest/run.html).</i>

# Test your server
Enter your server's domain in your browser and see if it works! The connection should always automatically upgrade to HTTPS; you can test this by explicitly using HTTP:
```
http://your.domain.com
Note the use of 'http://' instead of 'https://'!
```
If TailsX loads and automatically is using HTTPS, then you have set it up properly.

I <b>strongly</b> recommend testing your server's SSL/TLS certificate next using [SSLLabs](https://www.ssllabs.com/ssltest/index.html). You should recieve an <b>A+</b> rating with the provided nginx & certbot setup.

If you'd like, you can add your server to the list of public instances! Fork the repo linked [here](https://github.com/Extravi/tailsx) and add your own instance to the instance list (found in `/README.md`).
