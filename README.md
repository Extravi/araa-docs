# tailsx-docs

[![Counter](https://visitor-badge.laobi.icu/badge?page_id=Extravi.tailsxdocs)](https://github.com/Extravi/tailsx-docs)

To write this guide, I was using Ubuntu Server 22.04 LTS. This guide assumes you're using sudo or root.

Install required packages:

```bash
apt install nginx nginx-extras python3-pip certbot python3-certbot-nginx gunicorn
```

Clone TailsX:

```bash
git clone https://github.com/Extravi/tailsx.git
```

Configure opensearch.xml by replacing `http://127.0.0.1:5000/` with `https://tailsx.yourdomain.com/` make sure to replace `http://` with `https://`:

```bash
cd tailsx/
cd static/
mv opensearch.xml.example opensearch.xml
nano opensearch.xml
```

Once you've done that, cd back into the TailsX directory and install the required packages:

```bash
cd ~/tailsx
pip install flask lxml bs4
```

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

Obtain an SSL/TLS certificate, but before doing so, make sure you have an A record pointed to your server for that domain:
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

Now cd back into the TailsX directory:
```bash
cd ~/tailsx
```

Run this command to start TailsX:
```bash
gunicorn -w 4 __init__:app
```
