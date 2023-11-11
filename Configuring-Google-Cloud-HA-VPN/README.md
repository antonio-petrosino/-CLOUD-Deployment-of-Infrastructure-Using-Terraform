# HOW TO - Configuring Google Cloud HA VPN



![alt text](https://cdn.qwiklabs.com/VLAXpKCCD3vR0NwjlabV3gVp9Zzf1PPu7D91KCm9VY4%3D)


## Create a GLOBAL VPC with two custom subnetworks

Configure the VPC as follows. 

```bash
gcloud compute networks create vpc-demo --subnet-mode custom

gcloud compute networks subnets create vpc-demo-subnet1 --network vpc-demo --range 10.1.1.0/24 --region "REGION1"

gcloud compute networks subnets create vpc-demo-subnet2 --network vpc-demo --range 10.2.1.0/24 --region "REGION1"
```

Add some firewall rules. 

```bash
gcloud compute firewall-rules create vpc-demo-allow-custom --network vpc-demo   --allow tcp:0-65535,udp:0-65535,icmp --source-ranges 10.0.0.0/8

gcloud compute firewall-rules create vpc-demo-allow-ssh-icmp --network vpc-demo --allow tcp:22,icmp
```
Then, create two different instances.

```bash
gcloud compute instances create vpc-demo-instance1 --machine-type=e2-medium --zone "REGION1" --subnet vpc-demo-subnet1

gcloud compute instances create vpc-demo-instance2 --machine-type=e2-medium --zone "REGION1" --subnet vpc-demo-subnet2
```

## Simulated on-premise env

Similar process of the GLOBAL VPC. 

```bash
gcloud compute networks create on-prem --subnet-mode custom

gcloud compute networks subnets create on-prem-subnet1 --network on-prem --range 192.168.1.0/24 --region "REGION2"

gcloud compute firewall-rules create on-prem-allow-custom --network on-prem  --allow tcp:0-65535,udp:0-65535,icmp --source-ranges 192.168.0.0/16

gcloud compute firewall-rules create on-prem-allow-ssh-icmp --network on-prem --allow tcp:22,icmp

gcloud compute instances create on-prem-instance1 --machine-type=e2-medium --zone zone_name --subnet on-prem-subnet1
```


## Set up HA VPN gateway

Create the HA VPN gateway.

```bash
gcloud compute vpn-gateways create vpc-demo-vpn-gw1 --network vpc-demo --region "REGION1"
```
```bash
gcloud compute vpn-gateways create on-prem-vpn-gw1 --network on-prem --region "REGION2"
```

Then, the cloud router.

```bash
gcloud compute routers create vpc-demo-router1 --region "REGION1" --network vpc-demo --asn 65001
```
```bash
gcloud compute routers create on-prem-router1 --region "REGION2" --network on-prem --asn 65002
```

Now, the tunnels. 
```bash
gcloud compute vpn-tunnels create vpc-demo-tunnel0 --peer-gcp-gateway on-prem-vpn-gw1 --region "REGION1" --ike-version 2 --shared-secret PERSONAL_PASSWORD --router vpc-demo-router1 --vpn-gateway vpc-demo-vpn-gw1 --interface 0

gcloud compute vpn-tunnels create vpc-demo-tunnel1 --peer-gcp-gateway on-prem-vpn-gw1 --region "REGION1" --ike-version 2 --shared-secret PERSONAL_PASSWORD --router vpc-demo-router1 --vpn-gateway vpc-demo-vpn-gw1 --interface 1
```

```bash
gcloud compute vpn-tunnels create on-prem-tunnel0 --peer-gcp-gateway vpc-demo-vpn-gw1 --region "REGION2" --ike-version 2 --shared-secret PERSONAL_PASSWORD --router on-prem-router1 --vpn-gateway on-prem-vpn-gw1 --interface 0

gcloud compute vpn-tunnels create on-prem-tunnel1 --peer-gcp-gateway vpc-demo-vpn-gw1 --region "REGION2" --ike-version 2 --shared-secret PERSONAL_PASSWORD --router on-prem-router1 --vpn-gateway on-prem-vpn-gw1 --interface 1

```

# Set up BGP peering for each tunnel

Peering for each VPN tunnel by adding interface and BPG peer for tunnel0. 

1. vpc-demo network (Tunnel0 and Tunnel1):
    

```bash
gcloud compute routers add-interface vpc-demo-router1 --interface-name if-tunnel0-to-on-prem --ip-address 169.254.0.1 --mask-length 30 --vpn-tunnel vpc-demo-tunnel0 --region "REGION1"

gcloud compute routers add-bgp-peer vpc-demo-router1 --peer-name bgp-on-prem-tunnel0 --interface if-tunnel0-to-on-prem --peer-ip-address 169.254.0.2 --peer-asn 65002 --region "REGION1"
```

```bash
gcloud compute routers add-interface vpc-demo-router1 --interface-name if-tunnel1-to-on-prem --ip-address 169.254.1.1 --mask-length 30 --vpn-tunnel vpc-demo-tunnel1 --region "REGION1"

gcloud compute routers add-bgp-peer vpc-demo-router1 --peer-name bgp-on-prem-tunnel1 --interface if-tunnel1-to-on-prem --peer-ip-address 169.254.1.2 --peer-asn 65002 --region "REGION1"
```

2. on-prem network (Tunnel0 and Tunnel1):

```bash
gcloud compute routers add-interface on-prem-router1 --interface-name if-tunnel0-to-vpc-demo --ip-address 169.254.0.2 --mask-length 30 --vpn-tunnel on-prem-tunnel0 --region "REGION2"

gcloud compute routers add-bgp-peer on-prem-router1 --peer-name bgp-vpc-demo-tunnel0 --interface if-tunnel0-to-vpc-demo --peer-ip-address 169.254.0.1 --peer-asn 65001 --region "REGION2"
```

```bash
gcloud compute routers add-interface  on-prem-router1 --interface-name if-tunnel1-to-vpc-demo --ip-address 169.254.1.2 --mask-length 30 --vpn-tunnel on-prem-tunnel1 --region "REGION2"

gcloud compute routers add-bgp-peer  on-prem-router1 --peer-name bgp-vpc-demo-tunnel1 --interface if-tunnel1-to-vpc-demo --peer-ip-address 169.254.1.1 --peer-asn 65001 --region "REGION2"
```

## RemoteVPC firewall rules 

Allow network traffic

```bash
gcloud compute firewall-rules create vpc-demo-allow-subnets-from-on-prem --network vpc-demo --allow tcp,udp,icmp --source-ranges 192.168.1.0/24

gcloud compute firewall-rules create on-prem-allow-subnets-from-vpc-demo --network on-prem --allow tcp,udp,icmp --source-ranges 10.1.1.0/24,10.2.1.0/24
```

List all VPN tunnels

```bash
gcloud compute vpn-tunnels list


gcloud compute vpn-tunnels describe vpc-demo-tunnel0 --region "REGION1"
```

Finally, verify the configuration by ping some terminal in different networks.

Furthermore, enable global routing: 

```bash
gcloud compute networks update vpc-demo --bgp-routing-mode GLOBAL

gcloud compute networks describe vpc-demo
```


## Clean up env

```bash
gcloud compute vpn-tunnels delete on-prem-tunnel0  --region "REGION2"

gcloud compute vpn-tunnels delete vpc-demo-tunnel1  --region "REGION1"

gcloud compute vpn-tunnels delete on-prem-tunnel1  --region "REGION2"

gcloud compute routers remove-bgp-peer vpc-demo-router1 --peer-name bgp-on-prem-tunnel0 --region "REGION1"

gcloud compute routers remove-bgp-peer vpc-demo-router1 --peer-name bgp-on-prem-tunnel1 --region "REGION1"

gcloud compute routers remove-bgp-peer on-prem-router1 --peer-name bgp-vpc-demo-tunnel0 --region "REGION2"

gcloud compute routers remove-bgp-peer on-prem-router1 --peer-name bgp-vpc-demo-tunnel1 --region "REGION2"

gcloud compute  routers delete on-prem-router1 --region "REGION2"

gcloud compute  routers delete vpc-demo-router1 --region "REGION1"

gcloud compute vpn-gateways delete vpc-demo-vpn-gw1 --region "REGION1"

gcloud compute vpn-gateways delete on-prem-vpn-gw1 --region "REGION2"

gcloud compute instances delete vpc-demo-instance1 --zone "REGION1"

gcloud compute instances delete vpc-demo-instance2 --zone "REGION1"

gcloud compute instances delete on-prem-instance1 --zone "REGION2"

gcloud compute firewall-rules delete vpc-demo-allow-custom

gcloud compute firewall-rules delete on-prem-allow-subnets-from-vpc-demo

gcloud compute firewall-rules delete on-prem-allow-ssh-icmp

gcloud compute firewall-rules delete on-prem-allow-custom

gcloud compute firewall-rules delete vpc-demo-allow-subnets-from-on-prem

gcloud compute firewall-rules delete vpc-demo-allow-ssh-icmp

gcloud compute networks subnets delete vpc-demo-subnet1 --region "REGION1"

gcloud compute networks subnets delete vpc-demo-subnet2 --region "REGION1"

gcloud compute networks subnets delete on-prem-subnet1 --region "REGION2"

gcloud compute networks delete vpc-demo

gcloud compute networks delete on-prem
```