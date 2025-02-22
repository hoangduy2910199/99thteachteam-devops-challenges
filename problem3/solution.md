# Scenario 1: Nginx is overloaded due to high traffic or under DDOS attack.
## Impact: Nginx may be killed to free up memory or the service will be hang and return 5xx error.
## Troubleshoot
1. Check Nginx error log
    ```bash
    sudo tail -f /var/log/nginx/error.log
    sudo tail -f /var/log/nginx/access.log
2. Check cloudwatch metric

## Resolution:
1. Implement auto-scaling for EC2 which hosting Nginx.
2. Enable AWS Shield / WAF to mitigate DDoS attacks.

# Scenario 2: other process consumming high memory.
## Impact: Nginx may be killed to free up memory or the service will be hang and return 5xx error.
## Troubleshoot
1. Check current memory usage in the VM to ensure if the it's real memory pressure.
    ```bash
    free -m
2. Check top memory usage to find out what cosuming high mem.
    ```bash
    ps aux --sort=-%mem | head -10
## Resolution:
1. If there are any process running unexpectedly, escalating related teams to kill and disable it(if service).
2. work-around: enable swap if swap pace too small or no swap.
    ```bash
    sudo fallocate -l 2G /swapfile
    sudo chmod 600 /swapfile
    sudo mkswap /swapfile
    sudo swapon /swapfile
3. enable auto-restart for Ngnix after killed.
    ```bash
    [Service]
    Restart=always
    RestartSec=5s
# Scenario 3: there is error in upstream server or network makes requests stuck in Nginx server.
## Impact: Network stuck or application error causes requests cannot complete and queued in Nginx. It leads to high memory consumed and Nginx may be killed to free up memory or the service will be hang and return 5xx error.
## Troubleshoot:
1. Check Nginx error log, look for any error message such as: request timeout, connection refused, . . .
    ```bash
    sudo tail -f /var/log/nginx/error.log
2. check DNS resolutoin andnetwork accessibility: 
    ```bash
    nslookup <domain_name>
    telnet <backend_port> 
3. Check health of backend:
    ```bash
    curl -v ""
    check backend service is running or not - systemctl status <service_name>.
## resolution:
1. if DNS/network issue, check DNS server, NSG or firewall. Escalate to network team if neccessary.
2. If service hange, restart backend service to recover first.
3. if backend overloaded, increase resource configuration or scale up backend. -> for long term solution, apply auto scaling.
4. if application issue -> escalate to application team for investigation.
5. Reduce timeout configuration in Nginx to avoid hanging requests.
    ```bash
    proxy_connect_timeout 5s;
    proxy_send_timeout 10s;
    proxy_read_timeout 10s;
    send_timeout 10s;
6. Configure proactive alert for early issue detection.
        