# Project 3: Creating A Load Balancer using Google Cloud CLI (Command Line Interface)

Basic LoadBalancer understanding:
Dr. Burns - "A load balancer is a useful way to distribute traffic to multiple servers in order to reduce the workload on any one 
specific server. Load balancers can be used in all sorts of situations, such as managing email, web, or other internet traffic."

A load balancer acts as the “traffic cop” sitting in front of your servers and routing client requests across all 
servers capable of fulfilling those requests in a manner that maximizes speed and capacity utilization and ensures
that no one server is overworked, which could degrade performance. If a single server goes down, the load balancer
redirects traffic to the remaining online servers. When a new server is added to the server group, the load balancer 
automatically starts to send requests to it.

>> - Maximize server throughput and minimize Latency
>> - Handle the requests by scaling the system

A single server has limited resources and limited throughput to accomodate requests from all clients. This may cause the server to overload and cause slowness and failure.

To handle this we typically scale the system: 
1. Vertical scaling - increasing the power of server.
   An increase in resources is limited so we may want to choose the option of scaling the system by analysing the need.
2. Horiztontal scaling - popular option
- Add more machines or servers to the system, three servers instead of one, a lot more requests can be handled from clients. 
- Must ensure that requests are being processed by the server in a balanced way.
  > Horizontal scaling > add servers > load balancer will make use of new resources to improve the overall throughput > reduced latency

**Client >>>> Load Balancer >>>>> Server**
- Load Balancer is basically another server that sits between client and server to balance the load and requests from client to the servers. 
- Redirect clients requests to servers in balanced way
- Balance workloads
- Maintain even distribution 
- Main purpose is to redirect the requests in a balanced way so that none of the singles server is overloaded with requests from clients.

*Reverse Proxy* - Load balancers can also work on behalf of the clients or servers 
- Not limited to clients or servers
- LB can be setup in between 
- Server >>>>> Database 
- Server >>>>> other servers
- DNS Load Balancer at the DNS layer of the website (e.g. dns query from browser will request the DNS LB to process the request and return the website)
- Software load balancers - cheap and highly customizable

**Server Selection Strategy**
ROUND ROBIN - goes through all the servers in a loop for even distribution of load amongst all servers
Customized Round Robin is called Weighted Round Robin where one powerful server is given more load and redirected to other servers
Server Traffic - balancing load based on traffice being served by the server 
LB pings the server to check the health and based on server speed response and resources and decides if or not to redirect traffic to those servers
IP Based - gets request from the clients and hashes # the ip address so specific server is serving a specific client
useful when servers are caching.

Based on need and situation, it is possible to use Multiple Load Balancers with different server selection strategies.

***PROXIES VS REVERSE PROXY***
> Proxy server - is the server acting as a gatekeeper between the private network and the public internet.

***Forward proxy server*** - acts as a guardian/gatekeeper safeguarding the network by reguating traffic blocking harmful websites, masking client IP addresses, 
logging user activity, bypassing content restrictions and commonly used for web data collection. 
            *Benefits of Forward proxy servers (to protect clients)*
- Logs user activity
- logs what websites were visited by clients
- keeps track of what websites were visited and how long were they on those websites.
- bypass restricted contents. e.g. for schools and governments restrict some websites.
- improved speed by caching copies of websites frequently used. 

***Reverse Proxy server*** - regulates the traffic coming to the network. enhances security for servers by hiding their IP addresses, blocking malicious traffic, 
and implementing load balancing to distribute traffic evenly.
- creates a single POE to regulate incoming traffic to the servers.
           *Benefits of Reverse proxy (to protect servers)*
- increases the security on a private network by hiding IP addresses of the servers
- block malicious traffic such as DDOS attack. 
- where multiple servers are used, RP can act as load balancing to evenly distribute traffic to different servers to
reduce overwhelming traffic to one server causing slowness or failure. (traffic cop)

## Project 3: Creating a Load Balancer using Command Line Interface in GCP(Google Cloud Platform) - VM: Rocky Linux 8 
We will create servers **vweb01**, **vweb02** and **vweb03** to balance our load.
      *Purpose:*
- Use the gcloud command line to create a load balancer that distributes web (HTTP) traffic to three separate servers.
- Create three servers that would contain the exact same content: our wordpress website created in Rocky Linux 8.
- Any requests to the website would be proxied through the load balancer, which would then, like a traffic cop, 
direct HTTP traffic to the servers, as needed.

### Steps (I took using CLI in macOS Monterey 12.6.8)
#### 1. Check the List of authorized users
> gcloud auth list
Credentialed Accounts
ACTIVE  ACCOUNT
*       935733144260-compute@developer.gserviceaccount.com

To set the active account, run:
    $ gcloud config set account `ACCOUNT'

[core]
project = finalsys-gs
Your active configuration is: [default]

*Just a Note*
It is recommended to use service account but can authorize other login
*       935733144260-compute@developer.gserviceaccount.com
[bimalanemkul@rocky-2023 ~]$ gcloud auth login
and follow the instructions on the prompt.

You can run:
  $ gcloud config set account `ACCOUNT`
to switch accounts if necessary. But you may need to add another account if you want to use another account.
 - Check the authentication list
[bimalanemkul@rocky-2023 ~]$ gcloud auth list
-----------
    Save the project ID to a variable name PROJECT_ID by exporting the obtained project ID as an variable making it
    accessible to other commands and scripts
[bimalanemkul@rocky-2023 ~]$ export PROJECT_ID=$(gcloud config get-value project)

# 2. List the default zone and set the default zone
    Getting the default zone
[bimalanemkul@rocky-2023 ~]$ gcloud config list compute/zone
[compute]
zone (unset)

Your active configuration is: [default]
-----------
*Since the zone is (unset), modify and set the default zone. For me, its "us-central1-c"*
[bimalanemkul@rocky-2023 ~]$ gcloud config set compute/zone us-central1-c

# 3. List the Region and set the default Region
[bimalanemkul@rocky-2023 ~]$ gcloud config list compute/region
[compute]
region (unset)

Your active configuration is: [default]
---------
*Since the Region is (unset), modify and set the default Region. For me, its "us-central1-Iowa"*
[bimalanemkul@rocky-2023 ~]$ gcloud config set compute/region us-central1-Iowa
Check if the Region is updated,
[bimalanemkul@rocky-2023 ~]$ gcloud config list compute/region
[compute]
region = us-central1-Iowa

Your active configuration is: [default]

# 4. Create additional VM instance using CLI
We already have one VM named "rocky-2023". Create three other instances named "vweb01", "vweb02 and "vweb03"

*You may need authenticate your gmail account using ACCOUNT=your@gmail.com*
            $ gcloud config set account `ACCOUNT`  

**1st webserver: vweb01**
[bimalanemkul@rocky-2023 ~]$ gcloud compute instances create vweb01 --zone=us-central1-c --image-family=rocky-linux-8 --image-project=rocky-linux-cloud --tags=network-lb-tag --machine-type=e2-medium --metadata=startup-script='#!bin/bash 
dnf update -y
dnf install httpd -y
systemctl enable httpd
systemctl start httpd
echo "<h3>WebServer: vweb01</h3> | tee /var/www/html/index.html
'

Created [https://www.googleapis.com/compute/v1/projects/finalsys-gs/zones/us-central1-c/instances/vweb01].
NAME    ZONE           MACHINE_TYPE  PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP    STATUS
vweb01  us-central1-c  e2-medium                  10.128.0.5   34.30.120.120  RUNNING
[bimalanemkul@rocky-2023 ~]$ 

**2nd webserver: vweb02 (Note made this machine-type=e2-small)**
[bimalanemkul@rocky-2023 ~]$ gcloud compute instances create vweb02 --zone=us-central1-c --image-family=rocky-linux-8 --image-project=rocky-linux-cloud --tags=network-lb-tag --machine-type=e2-small --metadata=startup-script='#!bin/bash 
dnf update -y
dnf install httpd -y
systemctl enable httpd
systemctl start httpd
echo "<h3>WebServer: vweb01</h3> | tee /var/www/html/index.html
'

Created [https://www.googleapis.com/compute/v1/projects/finalsys-gs/zones/us-central1-c/instances/vweb02].
NAME    ZONE           MACHINE_TYPE  PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP  STATUS
vweb02  us-central1-c  e2-small                   10.128.0.6   34.71.62.93  RUNNING

**3rd webserver: vweb03 (Note made this machine-type=e2-small)**
[bimalanemkul@rocky-2023 ~]$ gcloud compute instances create vweb03 --zone=us-central1-c --image-family=rocky-linux-8 --image-project=rocky-linux-cloud --tags=network-lb-tag --machine-type=e2-small --metadata=startup-script='#!bin/bash 
dnf update -y
dnf install httpd -y
systemctl enable httpd
systemctl start httpd
echo "<h3>WebServer: vweb01</h3> | tee /var/www/html/index.html
'

Created [https://www.googleapis.com/compute/v1/projects/finalsys-gs/zones/us-central1-c/instances/vweb03].
NAME    ZONE           MACHINE_TYPE  PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP    STATUS
vweb03  us-central1-c  e2-small                   10.128.0.7   34.31.166.118  RUNNING

# 4. Create the Firewall rule to allow external traffic to our VM machine:
[bimalanemkul@rocky-2023 ~]$ gcloud compute firewall-rules create www-firewall-network-lb \
> --target-tags network-lb-tag --allow tcp:80
Creating firewall...⠹Created [https://www.googleapis.com/compute/v1/projects/finalsys-gs/global/firewalls/www-firewall-network-lb].
Creating firewall...done.                                                           
NAME                     NETWORK  DIRECTION  PRIORITY  ALLOW   DENY  DISABLED
www-firewall-network-lb  default  INGRESS    1000      tcp:80        False

# 5. Check the VM instances list (3 additional servers have been created):
[bimalanemkul@rocky-2023 ~]$ gcloud compute instances list
NAME        ZONE           MACHINE_TYPE  PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP     STATUS
rocky-2023  us-central1-c  e2-medium                  10.128.0.4   34.122.172.103  RUNNING
vweb01      us-central1-c  e2-medium                  10.128.0.5   34.30.120.120   RUNNING
vweb02      us-central1-c  e2-small                   10.128.0.6   34.71.62.93     RUNNING
vweb03      us-central1-c  e2-small                   10.128.0.7   34.31.166.118   RUNNING

# 6. Using Curl command to verify that each instance is running:
[bimalanemkul@rocky-2023 ~]$ curl http://10.128.0.4
[bimalanemkul@rocky-2023 ~]$ curl http://10.128.0.5
[bimalanemkul@rocky-2023 ~]$ curl http://10.128.0.6
[bimalanemkul@rocky-2023 ~]$ curl http://10.128.0.7

(Result shows the html content)

******Main Task: 1. Configure the Load Balancing Server******
# 1. Creating a static external IP address for the load balancer:
*I have used region=us-central1 but it can be modified according to specific requirements*
  (we can check the GCloud regions list using command *gcloud compute regions list*)

[bimalanemkul@rocky-2023 ~]$ gcloud compute addresses create network-lb-ip-1 --region=us-central1
Created [https://www.googleapis.com/compute/v1/projects/finalsys-gs/regions/us-central1/addresses/network-lb-ip-1].

# 2. Adding Legacy HTTP health check resource:
(Health checks determine which instances can receive new connections.This health check is a basic configuration and can be used to monitor the health of instances serving HTTP traffic) 
-- basic-check represents Health_check_name --

[bimalanemkul@rocky-2023 ~]$ gcloud compute http-health-checks create basic-check
Created [https://www.googleapis.com/compute/v1/projects/finalsys-gs/global/httpHealthChecks/basic-check].
NAME         HOST  PORT  REQUEST_PATH
basic-check        80    /

# 3. Add a target pool in the same region as the instances. Create a target pool named "www-pool" which is associated with an HTTP health check named "basic-check." This is required for the service to function:
[bimalanemkul@rocky-2023 ~]$ gcloud compute target-pools create www-pool --region=us-central1 --http-health-check basic-check
Created [https://www.googleapis.com/compute/v1/projects/finalsys-gs/regions/us-central1/targetPools/www-pool].
NAME      REGION       SESSION_AFFINITY  BACKUP  HEALTH_CHECKS
www-pool  us-central1  NONE                      basic-check

# 4. Now, we need to add all the new instances vweb01,vweb02 and vweb03 we created to the pool named "www-pool":
[bimalanemkul@rocky-2023 ~]$ gcloud compute target-pools add-instances www-pool --instances vweb01,vweb02,vweb03
Updated [https://www.googleapis.com/compute/v1/projects/finalsys-gs/regions/us-central1/targetPools/www-pool].

# 5. Next we need to forward the rules created to the load balancer and target pools
        *we can check the status of forwarding rule using*
        gcloud compute forwarding-rules describe www-rule --region=us-central1

There are no forwarding rules created right now. But we will create a new rule name "www-rule" to target the 'www-pool' using the following command:

[bimalanemkul@rocky-2023 ~]$ gcloud compute forwarding-rules create www-rule --region=us-central1 --ports=80 --target-pool=www-pool --address=network-lb-ip-1
Created [https://www.googleapis.com/compute/v1/projects/finalsys-gs/regions/us-central1/forwardingRules/www-rule].

**Load balancer is configured, Now we need to send traffic to our newly created VM instances**

**Send Traffic to the VM instaces**
1. check the www-rule forwarding rule used by load balancer: 

[bimalanemkul@rocky-2023 ~]$ gcloud compute forwarding-rules describe www-rule --region=us-central1
IPAddress: 34.171.121.209
IPProtocol: TCP
creationTimestamp: '2023-12-02T10:10:48.469-08:00'
description: ''
fingerprint: ZVVum_rqTv8=
id: '3703055092761939399'
kind: compute#forwardingRule
labelFingerprint: 42WmSpB8rSM=
loadBalancingScheme: EXTERNAL
name: www-rule
networkTier: PREMIUM
portRange: 80-80
region: https://www.googleapis.com/compute/v1/projects/finalsys-gs/regions/us-central1
selfLink: https://www.googleapis.com/compute/v1/projects/finalsys-gs/regions/us-central1/forwardingRules/www-rule
target: https://www.googleapis.com/compute/v1/projects/finalsys-gs/regions/us-central1/targetPools/www-pool

2. Access the external IP address:

IPADDRESS=$(gcloud compute forwarding-rules describe www-rule --region=us-central1 --format="json" | jq -r .IPAddress)
[bimalanemkul@rocky-2023 ~]$ IPADDRESS=$(gcloud compute forwarding-rules describe www-rule --region=us-central1 --format="json" | jq -r .IPAddress)

# jq: command not found - if this error is seen that means we need to install jq- command-line JSON processor in the system
[bimalanemkul@rocky-2023 ~]$ sudo dnf install jq

"After that, we should be able to the run the command, variable IPADDRESS will contain the IP address associated with the specified forwarding rule. We can then use this variable in subsequent commands or scripts as needed."
[bimalanemkul@rocky-2023 ~]$ IPADDRESS=$(gcloud compute forwarding-rules describe www-rule --region=us-central1 --format="json" | jq -r .IPAddress)
[bimalanemkul@rocky-2023 ~]$ 

Check the external IP Address
[bimalanemkul@rocky-2023 ~]$ echo $IPADDRESS
34.171.121.209

Then use the curl command to access the external IP address.
[bimalanemkul@rocky-2023 ~]$ curl 34.171.121.209
* Output: html codes for HTTP Server Test Page

3. Use curl command to access the external IP address, replacing IP_ADDRESS with an external IP address from the previous command:

[bimalanemkul@rocky-2023 ~]$ while true; do curl -m1 34.171.121.209; done
* while true; do ... done: This creates an infinite loop. The true command always returns a true value, so the loop continues indefinitely.
* curl -m1 $IPADDRESS: 
This command uses curl to make an HTTP request to the IP address stored in the $IPADDRESS variable. 
* The -m1 option sets a timeout of 1 second.

# a simple Bash loop that continuously executes the curl command, attempting to make HTTP requests to the IP address stored in the $IPADDRESS variable. The -m1 option for curl sets a timeout of 1 second, so if the connection or request takes longer than 1 second, curl will exit, and the loop will start another iteration.

We will need press Ctrl+C to stop the running command as it is running in a loop.

******Main Task: 2. Create an HTTP load balancer******
*From Dr. Burns Notes and Google Cloud Skills Boost Course:*
HTTP(S) Load Balancing is implemented on Google Front End (GFE). GFEs are distributed globally and operate together using Google's global network and control plane. You can configure URL rules to route some URLs to one set of instances and route other URLs to other instances.

Requests are always routed to the instance group that is closest to the user, if that group has enough capacity and is appropriate for the request. If the closest group does not have enough capacity, the request is sent to the closest group that does have capacity.

To set up a load balancer with a Compute Engine backend, your VMs need to be in an instance group. The managed instance group provides VMs running the backend servers of an external HTTP load balancer. For this lab, backends serve their own hostnames.

1. To create and HTTP load balancer - first we need to create a load balancer template in the backend:
[bimalanemkul@rocky-2023 ~]$ gcloud compute instance-templates create lb-backend-template --region=us-central1 --network=default --subnet=default --tags=allow-health-check --machine-type=e2-medium --image-family=rocky-linux-8 --image-project=rocky-linux-cloud --metadata=startup-script='#!/bin/bash
> dnf update -y
> dnf install httpd -y
> systemctl enable httpd
> systemctl start httpd
> vm_hostname="$(curl -H "Metadata-Flavor:Google" \
> http://169.254.269.254/computeMedtadata/v1/instancename)"
> echo "Page served from: $vm_hostname" | \
> tee /var/www/html/index.html
> systemctl restart httpd
> '
Created [https://www.googleapis.com/compute/v1/projects/finalsys-gs/global/instanceTemplates/lb-backend-template].
NAME                 MACHINE_TYPE  PREEMPTIBLE  CREATION_TIMESTAMP
lb-backend-template  e2-medium                  2023-12-02T12:07:14.104-08:00

In this command:
--image-family=rocky-linux-8 and --image-project=rocky-linux-cloud specify the Rocky Linux 8 image.
The startup-script section is modified to use dnf for package management, install and configure Apache HTTP Server (httpd), and retrieve the instance name using curl and jq.

2. Create a managed instance group based on the template:
Managed instance groups (MIGs) let you operate apps on multiple identical VMs. You can make your workloads scalable and highly available by taking advantage of automated MIG services, including: autoscaling, autohealing, regional (multiple zone) deployment, and automatic updating.

[bimalanemkul@rocky-2023 ~]$ gcloud compute instance-groups managed create lb-backend-group \
> --template=lb-backend-template --size=2 --zone=us-central1-c
Created [https://www.googleapis.com/compute/v1/projects/finalsys-gs/zones/us-central1-c/instanceGroupManagers/lb-backend-group].
NAME              LOCATION       SCOPE  BASE_INSTANCE_NAME  SIZE  TARGET_SIZE  INSTANCE_TEMPLATE    AUTOSCALED
lb-backend-group  us-central1-c  zone   lb-backend-group    0     2            lb-backend-template  no

lb-backend-group: Specifies the name of the managed instance group.
--base-instance-name: Specifies the base name for the instances in the group. The instances will be named with this base name followed by a hyphen and a unique identifier.
--template=lb-backend-template: Specifies the instance template to use for creating instances in the group.
--size=2: Specifies the initial number of instances to create in the group. Adjust this value based on your requirements.
--zone=us-central1-c: Specifies the zone in which the managed instance group will be created. Replace us-central1-c with the appropriate zone for your deployment.

3. Create the fw-allow-health-check firewall rule.

[bimalanemkul@rocky-2023 ~]$ gcloud compute firewall-rules create fw-allow-health-check \
> --network=default \
> --action=allow \
> --direction=ingress \
> --source-ranges=130.211.0.0/22,35.191.0.0/16 \
> --target-tags=allow-health-check \
> --rules=tcp:80
Creating firewall...⠹Created [https://www.googleapis.com/compute/v1/projects/finalsys-gs/global/firewalls/fw-allow-health-check].
Creating firewall...done.                                                      
NAME                   NETWORK  DIRECTION  PRIORITY  ALLOW   DENY  DISABLED
fw-allow-health-check  default  INGRESS    1000      tcp:80        False

Note: The ingress rule allows traffic from the Google Cloud health checking systems (130.211.0.0/22 and 35.191.0.0/16).
gcloud compute firewall-rules create fw-allow-health-check: This is the main command to create a firewall rule in the Google Cloud Compute service. It specifies the name of the firewall rule as fw-allow-health-check.
--network=default: Specifies the network for which the firewall rule is created. In this case, it's the default network.
--action=allow: Sets the action for the firewall rule to allow. This means that incoming traffic meeting the specified conditions will be allowed.
--direction=ingress: Specifies that the firewall rule applies to incoming traffic (ingress).
--source-ranges=130.211.0.0/22,35.191.0.0/16: Defines the source IP ranges from which incoming traffic is allowed. In this case, it allows traffic from the specified Google Cloud IP ranges.
--target-tags=allow-health-check: Associates the firewall rule with instances that have the specified target tags. In this case, instances with the tag allow-health-check will be affected by this rule.
--rules=tcp:80: Specifies the protocol and port for which the rule applies. In this case, it allows TCP traffic on port 80.

4. Now that the instances are up and running, set up a global static external IP address that your customers use to reach your load balancer:
[bimalanemkul@rocky-2023 ~]$ gcloud compute addresses create lb-ipv4-1 --ip-version=IPV4 --global
Created [https://www.googleapis.com/compute/v1/projects/finalsys-gs/global/addresses/lb-ipv4-1].

(After running this command, you will have created a static external IPv4 address named lb-ipv4-1 in the global scope. You can use this IP address when configuring your load balancer, associating it with the frontend configuration to handle incoming traffic.)

gcloud compute addresses create lb-ipv4-1: This is the main command to create an external IP address in the Google Cloud Compute service. It specifies the name of the IP address as lb-ipv4-1. You can replace this with your desired name.
--ip-version=IPV4: Specifies the IP version for the address. In this case, it's set to IPv4. You might also see --ip-version=IPV6 for IPv6 addresses.
--global: Specifies that the IP address is global. Global IP addresses are not tied to a specific region and can be used for global load balancing.

Check to see if the IPV4 address was reserved. Note it down:
[bimalanemkul@rocky-2023 ~]$ gcloud compute addresses describe lb-ipv4-1 --format="get(address)" --global
34.144.234.84

Above command is used to retrieve information about a specific external IP address. In this case, the lb-ipv4-1 is specified, and the --format flag is used to extract the address from the output.

5. Create a health check for the load balancer:
[bimalanemkul@rocky-2023 ~]$ gcloud compute health-checks create http http-basic-check --port 80
Created [https://www.googleapis.com/compute/v1/projects/finalsys-gs/global/healthChecks/http-basic-check].
NAME              PROTOCOL
http-basic-check  HTTP

*the health check is named http-basic-check, and it is configured for HTTP health checking on port 80.*

6. Create a backend service:
[bimalanemkul@rocky-2023 ~]$ gcloud compute backend-services create web-backend-service \
> --protocol=HTTP \
> --port-name=http \
> --health-checks=http-basic-check \
> --global
Created [https://www.googleapis.com/compute/v1/projects/finalsys-gs/global/backendServices/web-backend-service].
NAME                 BACKENDS  PROTOCOL
web-backend-service            HTTP

* Command is used to create a backend service in Google Cloud Platform. In this specific example, the backend service is named web-backend-service, and it is configured for HTTP traffic. * 

7. Add your instance group as the backend to the backend service:
[bimalanemkul@rocky-2023 ~]$ gcloud compute backend-services add-backend web-backend-service --instance-group=lb-backend-group --instance-group-zone=us-central1-c --global
Updated [https://www.googleapis.com/compute/v1/projects/finalsys-gs/global/backendServices/web-backend-service].

*the backend service is named web-backend-service, and instances from the instance group lb-backend-group are being added.*

8. Create a URL map to route the incoming requests to the default backend service:
[bimalanemkul@rocky-2023 ~]$ gcloud compute url-maps create web-map-http \
> --default-service web-backend-service
Created [https://www.googleapis.com/compute/v1/projects/finalsys-gs/global/urlMaps/web-map-http].
NAME          DEFAULT_SERVICE
web-map-http  backendServices/web-backend-service

*the URL map is named web-map-http, and it is configured to use the web-backend-service as the default service.*
Note: URL map is a Google Cloud configuration resource used to route requests to backend services or backend buckets. For example, with an external HTTP(S) load balancer, you can use a single URL map to route requests to different destinations based on the rules configured in the URL map:
Requests for https://example.com/video go to one backend service.
Requests for https://example.com/audio go to a different backend service.
Requests for https://example.com/images go to a Cloud Storage backend bucket.
Requests for any other host and path combination go to a default backend service.

9. Create a target HTTP proxy to route requests to your URL map:
[bimalanemkul@rocky-2023 ~]$ gcloud compute target-http-proxies create http-lb-proxy \
> --url-map web-map-http
Created [https://www.googleapis.com/compute/v1/projects/finalsys-gs/global/targetHttpProxies/http-lb-proxy].
NAME           URL_MAP
http-lb-proxy  web-map-http

*the target HTTP proxy is named http-lb-proxy, and it is configured to use the web-map-http URL map.*

10. Create a global forwarding rule to route incoming requests to the proxy:
[bimalanemkul@rocky-2023 ~]$ gcloud compute forwarding-rules create http-content-rule \
> --address=lb-ipv4-1 \
> --global \
> --target-http-proxy=http-lb-proxy \
> --ports=80
Created [https://www.googleapis.com/compute/v1/projects/finalsys-gs/global/forwardingRules/http-content-rule].

* the forwarding rule is named http-content-rule, and it is configured to forward HTTP traffic to the http-lb-proxy target HTTP proxy. Additionally, it associates the rule with the global external IP address named lb-ipv4-1 and specifies that it should handle traffic on port 80.

******Final Task 3. Testing traffic sent to your instances******
a. In the Google Cloud console, from the Navigation menu, go to Network services > Load balancing.
b. Click on the load balancer that you just created (web-map-http).
c. In the Backend section, click on the name of the backend and confirm that the VMs are Healthy. If they are not healthy, wait a few moments and try reloading the page.
d. When the VMs are healthy, test the load balancer using a web browser, going to http://IP_ADDRESS/, replacing IP_ADDRESS with the load balancer's IP address.
*This may take three to five minutes. If you do not connect, wait a minute, and then reload the browser.*

Your browser should render a page with content showing the name of the instance that served the page, along with its zone (for example, Page served from: lb-backend-group-xxxx).



-----------
[^] Footnote
UseFul Resources: (Google account)
https://www.cloudskillsboost.google/catalog_lab/1034
https://www.cloudskillsboost.google/course_sessions/6628477/labs/403395
https://www.youtube.com/watch?v=ZcNaOuxcuyA
https://www.youtube.com/watch?v=RXXRguaHZs0
https://www.nginx.com/resources/glossary/reverse-proxy-vs-load-balancer/

Special Thanks to UKY Professor Dr. Sean Burns for giving us the opportunity to get a hands on experience with Linux and other
areas of computing and technology. 
