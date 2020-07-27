# cloud-proxy
cloud-proxy creates multiple cloud instances and then starts local socks proxies using SSH. After exiting, the droplets are deleted.

### Warning
This tool will deploy as many droplets as you desire, and will make a best effort to delete them after use. However, you are ultimately going to pay the bill for these droplets, and it is up to you, and you alone to ensure they actually get deleted.

### Install
Git clone it and do a `go build .`
Download terraform [here](https://www.terraform.io/downloads.html).  
Currently the only supported and tested OS is Linux:

```
$ ./cloud-proxy
```
### Usage
```
Usage of ./cloud-proxy:
  -aws
    	Use AWS as provider
  -awsRegions string
    	Comma separated list of regions to deploy droplets to, defaults to all. (default "*")
  -count int
    	Amount of droplets to deploy (default 5)
  -do
    	Use DigitalOcean as provider
  -doRegions string
    	Comma separated list of regions to deploy droplets to, defaults to all. (default "*")
  -force
    	Bypass built-in protections that prevent you from deploying more than 50 droplets
  -key-location string
    	SSH key location (default "~/.ssh/id_rsa")
  -name string
    	Droplet name prefix (default "cloud-proxy")
  -start-tcp int
    	TCP port to start first proxy on and increment from (default 55555)
  -v	Print version and exit
```

### Getting Started
To use cloud-proxy with DO you will need to have a DO API token, you can get one [here](https://cloud.digitalocean.com/settings/api/tokens).
To use cloud-proxy with AWS you will need to have an Access and Secret key.

###### DO

Create an actually good SSH key with:

```
ssh-keygen -t ed25519 -a 100 -f ~/.ssh/id_rsa
```

Add this key to DO. Following it's addition, you will be able to grab the MD5 associated with the key. Otherwise do

```
ssh-keygen -l -E md5 -f ~/.ssh/{{filename}}
```
Create a `secrets.tfvars` file that will contain the API tokens and information for SSH. The file should look like the following:
```
do_token = "YOUR_DO_TOKEN"
do_ssh_fingerprint = "YOUR:SSH:FINGERPRINT"
aws_access_key = "YOUR_ACCESS_KEY"
aws_secret_key = "YOUR_SECRET_KEY"
aws_key_name = "SSH_KEY_NAME"
```

Now you may create some proxies:
```
$ ./cloud-proxy -do -aws -count 15
```

###### TIPS and Troubleshooting

* Do in a screen session 
* Double check that your has proper permissions in ~/.ssh by `chmod 400`
* If you are seeing failed to connect or something: Try to connecting to a provisioned server with your key quickly, killing everything, and trying again. I am not really sure why this works.

When you are finished using your proxies, use CTRL-C to interrupt the program, cloud-proxy will catch the interrupt and delete the instances.


### Example Output
```
$ ./cloud-proxy -token <my_token> -key <my_fingerprint>
==> Info: Droplets deployed. Waiting 100 seconds...
==> Info: SSH proxy started on port 55555 on droplet name: cloud-proxy-1 IP: <IP>
==> Info: SSH proxy started on port 55556 on droplet name: cloud-proxy-2 IP: <IP>
==> Info: SSH proxy started on port 55557 on droplet name: cloud-proxy-3 IP: <IP>
==> Info: SSH proxy started on port 55558 on droplet name: cloud-proxy-4 IP: <IP>
==> Info: SSH proxy started on port 55559 on droplet name: cloud-proxy-5 IP: <IP>
==> Info: proxychains config
socks5 127.0.0.1 55555
socks5 127.0.0.1 55556
socks5 127.0.0.1 55557
socks5 127.0.0.1 55558
socks5 127.0.0.1 55559
==> Info: socksd config
"upstreams": [
{"type": "socks5", "address": "127.0.0.1:55555"},
{"type": "socks5", "address": "127.0.0.1:55556"},
{"type": "socks5", "address": "127.0.0.1:55557"},
{"type": "socks5", "address": "127.0.0.1:55558"},
{"type": "socks5", "address": "127.0.0.1:55559"}
]
==> Info: Please CTRL-C to destroy droplets
^C==> Info: Deleted droplet name: cloud-proxy-1
==> Info: Deleted droplet name: cloud-proxy-2
==> Info: Deleted droplet name: cloud-proxy-3
==> Info: Deleted droplet name: cloud-proxy-4
==> Info: Deleted droplet name: cloud-proxy-5
```

