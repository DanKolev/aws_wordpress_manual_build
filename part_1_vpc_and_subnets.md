# Create VPC and Subnets

![Subnets](https://github.com/DanKolev/aws_wordpress_manual_build/blob/main/diagrams/1.vpc_with-subnetting.png)


# VPC 

Create VPC $Name

- IP: 10.64.0.0/24

- No. of subnets:   16

- No. hosts/subnet: 11 (16-5)



AWS Reserved IPs

- AWS Reserved IPs = 5

  - 0.0: Network address.

  - 0.1: Reserved by AWS for the VPC router.

  - 0.2: Reserved by AWS. The IP address of the DNS server is the base of the VPC network range plus two.

  - 0.3: Reserved by AWS for future use.

  - 0.255: Network broadcast address.


Spare capacity range for future use
- 10.64.0.144 - 10.64.0.255

# Subnets

Subnet format: 

- db – app – pub

Name and CIDR for availability zones

- AZA

  - sn-db-A 10.64.0.0/28 AZA IPv6 00 → IPv6 custom value

  - sn-app-A 10.64.0.16/28 AZA IPv6 01

  - sn-web-A 10.64.0.32/28 AZA IPv6 02


- AZB

  - sn-db-B 10.64.0.48/28 AZB IPv6 03

  - sn-app-B 10.64.0.64/28 AZB IPv6 04

  - sn-web-B 10.64.0.80/28 AZB IPv6 05


- AZC

  - sn-db-C 10.64.0.96/28 AZC IPv6 06

  - sn-app-C 10.64.0.112/28 AZC IPv6 07

  - sn-web-C 10.64.0.128/28 AZC IPv6 08


- Remember to enable auto assign ipv6 on every subnet you create.

IPv6 custom value

- Every VPC can be allocated with  /56 IPv6 CIDR. Every subnet within that VPC can optionally be allocated with /64 IPv6 CIDR. There are 256 /64 ranges within /56 
IPv6 CIDR. A hexadecimal value ranging from 00 to FF provides 256 /64possible values.

# Routing tables

Route tables format:

- lab-vpc1-rt-web

- lab-vpc1-rt-app

- lab-vpc1-rt-db

Create RT and associate with web subnets.

Create routes for IPv4/6 to GW.

Enable auto-assign IPv4 for the web subnets.






