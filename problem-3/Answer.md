# First though.

With 99% disk usage, we might still be able to SSH into the NGINX VM. First step is to get in and check if the disk is actually full:

```bash
# Check disk usage
sudo df -h /
```
If the disk is full correctly, then need to go the next step 

## Checking log rotating 

Since this VM only runs NGINX as a load balancer, logs are probably the culprit.

```bash
# Check log sizes
sudo du -sh /var/log/
sudo du -sh /var/log/nginx/
```

If `/var/log` is eating the space, the issues are probably:
* NGINX logs aren't rotating correctly, or the the speed of logs is increasing unsually (`access.log`, `error.log`)
* System logs growing out of control ( Mostly nope happen in this case )

## DDoS check
IF we can go to monitoring in cloud for this storage size of this VM for checking pattern of increasing, if this increasing is weird not continouly, we should check that if we are under DDOs or not, or it's the real traffic from end users.

Check some IP accessing pattern with the format log in current Nginx for seeing which top IPs calling in acess log.

## Impact

* NGINX stops logging
* 5xx errors for users
* VM could crash
* Other services affected

## Root Causes

* No log rotation
* DDoS flooding logs
* Monitoring set way too high (99% is too late with storage size is 66 GB)

## Resolve issue

### 1. Free up space immediately

```bash
# Truncate logs (better than rm since nginx has the files open)
sudo truncate -s 0 /var/log/nginx/access.log
sudo truncate -s 0 /var/log/nginx/error.log

# Check if it worked
sudo df -h /
```

### 2. Set up/Increase the frequently of  log rotation

```bash
# Edit logrotate config
sudo nano /etc/logrotate.d/nginx
```

Add this:
```
/var/log/nginx/*.log {
    daily
    missingok
    rotate 7 # Should decrease to 7 days instead of 30 days in case we will keep the storage size is 66gb.
    ...
}
```
### 3. Handle DDoS (If we figure out some patterns)

#### CloudFlare (best option if we use)
* Set Security Level to "Medium" 
* Turn on "Under Attack Mode" if needed

#### Block bad IPs on the server
```
for IP in $BAD_IPs; do
  sudo iptables -A INPUT -s $IP -j DROP
done
```

### 4. The other options 
* Increase the storage size if we detect that this log increasing because the surge of traffic 
* Integrate centralize cloud logging solution like rotating more frequenly and storing in GCS (cold storage ), ... S3  for checking latter.
* Setting threshold is lower 80% for 66 gb ~ 13 GB remaining, should be safe for tackling issues
