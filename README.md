# Azure VPN Virtual Network Gateway BGP lab

## Create resource group and vnets

Resource group, vnet, subnets:

```
rg=vngbgp
location=westeurope
az group create -n $rg -l $location
az network vnet create -g $rg -n vnet0 --address-prefix 10.0.0.0/16 --subnet-name GatewaySubnet --subnet-prefix 10.0.0.0/24
az network vnet create -g $rg -n vnet11 --address-prefix 10.11.0.0/16 --subnet-name GatewaySubnet --subnet-prefix 10.11.0.0/24
az network vnet create -g $rg -n vnet12 --address-prefix 10.12.0.0/16 --subnet-name GatewaySubnet --subnet-prefix 10.12.0.0/24
az network vnet create -g $rg -n vnet21 --address-prefix 10.21.0.0/16 --subnet-name GatewaySubnet --subnet-prefix 10.21.0.0/24
az network vnet create -g $rg -n vnet22 --address-prefix 10.22.0.0/16 --subnet-name GatewaySubnet --subnet-prefix 10.22.0.0/24
az network vnet subnet create -n VM --vnet-name vnet0 -g $rg --address-prefixes "10.0.1.0/24"
az network vnet subnet create -n VM --vnet-name vnet11 -g $rg --address-prefixes "10.11.1.0/24"
az network vnet subnet create -n VM --vnet-name vnet12 -g $rg --address-prefixes "10.12.1.0/24"
az network vnet subnet create -n VM --vnet-name vnet21 -g $rg --address-prefixes "10.21.1.0/24"
az network vnet subnet create -n VM --vnet-name vnet22 -g $rg --address-prefixes "10.22.1.0/24"
```

## Create VNGs

Two public IPs are created for each Virtual Network Gateway for active/active. The VNGs are created with BGP support:

```
# VNG 0
az network public-ip create -g $rg -n pip0a
az network public-ip create -g $rg -n pip0b
az network vnet-gateway create -g $rg -sku VpnGw1 --gateway-type Vpn --vpn-type Route-Based --vnet vnet0 -n vng0 --asn 65000 --public-ip-address pip0a pip0b --no-wait
# VNG 11
az network public-ip create -g $rg -n pip11a
az network public-ip create -g $rg -n pip11b
az network vnet-gateway create -g $rg -sku VpnGw1 --gateway-type Vpn --vpn-type Route-Based --vnet vnet11 -n vng11 --asn 65011 --public-ip-address pip11a pip11b --no-wait
# VNG 12
az network public-ip create -g $rg -n pip12a
az network public-ip create -g $rg -n pip12b
az network vnet-gateway create -g $rg -sku VpnGw1 --gateway-type Vpn --vpn-type Route-Based --vnet vnet12 -n vng12 --asn 65012 --public-ip-address pip11a pip11b --no-wait
# VNG 21
az network public-ip create -g $rg -n pip21a
az network public-ip create -g $rg -n pip21b
az network vnet-gateway create -g $rg -sku VpnGw1 --gateway-type Vpn --vpn-type Route-Based --vnet vnet21 -n vng21 --asn 65021 --public-ip-address pip21a pip21b --no-wait
# VNG 22
az network public-ip create -g $rg -n pip22a
az network public-ip create -g $rg -n pip22b
az network vnet-gateway create -g $rg -sku VpnGw1 --gateway-type Vpn --vpn-type Route-Based --vnet vnet22 -n vng22 --asn 65022 --public-ip-address pip22a pip22b --no-wait
```

## Create test Virtual Machines

We will create a VM in one vnet. The purpose is testing connectivity and being able to inspect the effective routing tables:

```
username=lab-user
password=YourSuper$ecret!  # Or better, git it from Key Vault
az vm create -n vm0 -g $rg --image ubuntults --admin-username $username --admin-password $password --public-ip-address vm0pip --vnet-name vnet0 --subnet VM --no-wait
az vm create -n vm11 -g $rg --image ubuntults --admin-username $username --admin-password $password --public-ip-address vm11pip --vnet-name vnet11 --subnet VM --no-wait
az vm create -n vm12 -g $rg --image ubuntults --admin-username $username --admin-password $password --public-ip-address vm12pip --vnet-name vnet12 --subnet VM --no-wait
az vm create -n vm21 -g $rg --image ubuntults --admin-username $username --admin-password $password --public-ip-address vm21pip --vnet-name vnet21 --subnet VM --no-wait
az vm create -n vm22 -g $rg --image ubuntults --admin-username $username --admin-password $password --public-ip-address vm22pip --vnet-name vnet22 --subnet VM --no-wait
```

## Create connections between the VNGs

```
password=YourSuper$ecret!  # Or better, get it from Key Vault
# vnet0 <-> vnet 11
az network vpn-connection create -g $rg --shared-key $password --enable-bgp -n 0to11 --vnet-gateway1 vng0 --vnet-gateway2 vng11
az network vpn-connection create -g $rg --shared-key $password --enable-bgp -n 11to0 --vnet-gateway1 vng11 --vnet-gateway2 vng0
# vnet0 <-> vnet 12
az network vpn-connection create -g $rg --shared-key $password --enable-bgp -n 0to12 --vnet-gateway1 vng0 --vnet-gateway2 vng12
az network vpn-connection create -g $rg --shared-key $password --enable-bgp -n 12to0 --vnet-gateway1 vng12 --vnet-gateway2 vng0
# vnet11 <-> vnet 21
az network vpn-connection create -g $rg --shared-key $password --enable-bgp -n 11to21 --vnet-gateway1 vng11 --vnet-gateway2 vng21
az network vpn-connection create -g $rg --shared-key $password --enable-bgp -n 21to11 --vnet-gateway1 vng21 --vnet-gateway2 vng11
# vnet11 <-> vnet 22
az network vpn-connection create -g $rg --shared-key $password --enable-bgp -n 11to22 --vnet-gateway1 vng11 --vnet-gateway2 vng22
az network vpn-connection create -g $rg --shared-key $password --enable-bgp -n 22to11 --vnet-gateway1 vng22 --vnet-gateway2 vng11
# vnet12 <-> vnet 21
az network vpn-connection create -g $rg --shared-key $password --enable-bgp -n 12to21 --vnet-gateway1 vng12 --vnet-gateway2 vng21
az network vpn-connection create -g $rg --shared-key $password --enable-bgp -n 21to12 --vnet-gateway1 vng21 --vnet-gateway2 vng12
# vnet12 <-> vnet 22
az network vpn-connection create -g $rg --shared-key $password --enable-bgp -n 12to22 --vnet-gateway1 vng12 --vnet-gateway2 vng22
az network vpn-connection create -g $rg --shared-key $password --enable-bgp -n 22to12 --vnet-gateway1 vng22 --vnet-gateway2 vng12

```

Verify connections:

```
az network vpn-connection list -g vngbgp -o table
ConnectionProtocol    ConnectionType    EgressBytesTransferred    EnableBgp    ExpressRouteGatewayBypass    IngressBytesTransferred    Location    Name    ProvisioningState    ResourceGroup    ResourceGuid                          RoutingWeight    UsePolicyBasedTrafficSelectors
--------------------  ----------------  ------------------------  -----------  ---------------------------  -------------------------  ----------  ------  -------------------  ---------------  ------------------------------------  ---------------  --------------------------------
IKEv2                 Vnet2Vnet         0                         True         False                        0                          westeurope  0to11   Succeeded            vngbgp           eb8a6af3-4050-44f1-a784-29a160085eb2  10               False
IKEv2                 Vnet2Vnet         0                         True         False                        0                          westeurope  0to12   Succeeded            vngbgp           b3a2c2b4-2736-4a20-b16b-3c11afd02697  10               False
IKEv2                 Vnet2Vnet         0                         True         False                        0                          westeurope  11to0   Succeeded            vngbgp           f03b1f8e-e29f-4ba7-8475-9b0601da32ee  10               False
IKEv2                 Vnet2Vnet         0                         True         False                        0                          westeurope  11to21  Succeeded            vngbgp           4e419f91-51ce-43e7-9753-96a128e5aeb7  10               False
IKEv2                 Vnet2Vnet         0                         True         False                        0                          westeurope  11to22  Succeeded            vngbgp           1c86de9b-6014-4bdf-8c8c-563989cbef16  10               False
IKEv2                 Vnet2Vnet         0                         True         False                        0                          westeurope  12to0   Succeeded            vngbgp           c9642a14-8721-459b-b4ca-0339f20cc847  10               False
IKEv2                 Vnet2Vnet         0                         True         False                        0                          westeurope  12to21  Succeeded            vngbgp           99ba99de-ff83-4b19-8390-4bdfab0c9fdf  10               False
IKEv2                 Vnet2Vnet         0                         True         False                        0                          westeurope  12to22  Succeeded            vngbgp           d87cfb38-36c7-486c-a711-0844c8fd0ab9  10               False
IKEv2                 Vnet2Vnet         0                         True         False                        0                          westeurope  21to11  Succeeded            vngbgp           7b202875-ac23-4940-8ed4-054fe9699607  10               False
IKEv2                 Vnet2Vnet         0                         True         False                        0                          westeurope  21to12  Succeeded            vngbgp           92a787c3-c9b4-44c3-962f-81fb069ca959  10               False
IKEv2                 Vnet2Vnet         0                         True         False                        0                          westeurope  22to11  Succeeded            vngbgp           6d072f29-4683-43b4-b32f-a70d7c902f1d  10               False
IKEv2                 Vnet2Vnet         0                         True         False                        0                          westeurope  22to12  Succeeded            vngbgp           e1443bce-0702-4fc9-92fd-121e9919b0da  10               False
```

## BGP topology: vnet0

vnet0 peers with vnet11 and vnet12. Note there is both iBGP and eBGP adjacencies:

```
az network vnet-gateway list-bgp-peer-status -g vngbgp -n vng0 -o table
Neighbor    ASN    State      ConnectedDuration    RoutesReceived    MessagesSent    MessagesReceived                                                                                                               ----------  -----  ---------  -------------------  ----------------  --------------  ------------------
10.12.0.5   65012  Connected  00:00:02.2554298     10                49              13
10.12.0.4   65012  Connected  00:02:33.5511715     10                10              11
10.11.0.5   65011  Connected  00:31:08.2667561     10                47              48
10.11.0.4   65011  Connected  00:31:08.2667561     10                44              51
10.0.0.4    65000  Connected  00:52:41.2022402     12                84              87
10.0.0.5    65000  Unknown                         0                 0               0
10.12.0.5   65012  Connected  00:02:48.0239933     10                11              12
10.12.0.4   65012  Connected  00:00:03.3545090     10                43              13
10.11.0.5   65011  Connected  00:31:18.7076810     10                46              50
10.11.0.4   65011  Connected  00:31:15.7448744     10                47              54
10.0.0.4    65000  Unknown                         0                 0               0
10.0.0.5    65000  Connected  00:52:41.1223392     12                85              86
```

Not sure what the unknown neighbors are, but they appear everywhere and do not seem to affect functionality.

Routing table:
 
```
az network vnet-gateway list-learned-routes -g vngbgp -n vng0 -o table
Network       Origin    SourcePeer    AsPath             Weight    NextHop
------------  --------  ------------  -----------------  --------  ---------
10.0.0.0/16   Network   10.0.0.4                         32768
10.11.0.4/32  Network   10.0.0.4                         32768
10.11.0.4/32  IBgp      10.0.0.5                         32768     10.0.0.5
10.11.0.5/32  Network   10.0.0.4                         32768
10.11.0.5/32  IBgp      10.0.0.5                         32768     10.0.0.5
10.11.0.0/16  EBgp      10.11.0.5     65011              32768     10.11.0.5
10.11.0.0/16  EBgp      10.11.0.4     65011              32768     10.11.0.4
10.11.0.0/16  IBgp      10.0.0.5      65011              32768     10.0.0.5
10.21.0.4/32  EBgp      10.11.0.4     65011              32768     10.11.0.4
10.21.0.4/32  IBgp      10.0.0.5      65011              32768     10.0.0.5
10.21.0.4/32  EBgp      10.11.0.5     65011              32768     10.11.0.5
10.21.0.4/32  EBgp      10.12.0.5     65012              32768     10.12.0.5
10.21.0.4/32  EBgp      10.12.0.4     65012              32768     10.12.0.4
10.21.0.5/32  EBgp      10.11.0.4     65011              32768     10.11.0.4
10.21.0.5/32  IBgp      10.0.0.5      65011              32768     10.0.0.5
10.21.0.5/32  EBgp      10.11.0.5     65011              32768     10.11.0.5
10.21.0.5/32  EBgp      10.12.0.5     65012              32768     10.12.0.5
10.21.0.5/32  EBgp      10.12.0.4     65012              32768     10.12.0.4
10.21.0.0/16  EBgp      10.11.0.4     65011-65021        32768     10.11.0.4
10.21.0.0/16  EBgp      10.11.0.5     65011-65021        32768     10.11.0.5
10.21.0.0/16  IBgp      10.0.0.5      65011-65021        32768     10.0.0.5
10.21.0.0/16  EBgp      10.12.0.5     65012-65021        32768     10.12.0.5
10.21.0.0/16  EBgp      10.12.0.4     65012-65021        32768     10.12.0.4
10.22.0.4/32  EBgp      10.11.0.4     65011              32768     10.11.0.4
10.22.0.4/32  EBgp      10.11.0.5     65011              32768     10.11.0.5
10.22.0.4/32  IBgp      10.0.0.5      65011              32768     10.0.0.5
10.22.0.4/32  EBgp      10.12.0.5     65012              32768     10.12.0.5
10.22.0.4/32  EBgp      10.12.0.4     65012              32768     10.12.0.4
10.22.0.5/32  EBgp      10.11.0.4     65011              32768     10.11.0.4
10.22.0.5/32  EBgp      10.11.0.5     65011              32768     10.11.0.5
10.22.0.5/32  IBgp      10.0.0.5      65011              32768     10.0.0.5
10.22.0.5/32  EBgp      10.12.0.5     65012              32768     10.12.0.5
10.22.0.5/32  EBgp      10.12.0.4     65012              32768     10.12.0.4
10.22.0.0/16  EBgp      10.11.0.5     65011-65022        32768     10.11.0.5
10.22.0.0/16  IBgp      10.0.0.5      65011-65022        32768     10.0.0.5
10.22.0.0/16  EBgp      10.11.0.4     65011-65022        32768     10.11.0.4
10.22.0.0/16  EBgp      10.12.0.5     65012-65022        32768     10.12.0.5
10.22.0.0/16  EBgp      10.12.0.4     65012-65022        32768     10.12.0.4
10.12.0.4/32  Network   10.0.0.4                         32768
10.12.0.4/32  IBgp      10.0.0.5                         32768     10.0.0.5
10.12.0.5/32  Network   10.0.0.4                         32768
10.12.0.5/32  IBgp      10.0.0.5                         32768     10.0.0.5
10.12.0.0/16  EBgp      10.12.0.5     65012              32768     10.12.0.5
10.12.0.0/16  EBgp      10.12.0.4     65012              32768     10.12.0.4
10.12.0.0/16  IBgp      10.0.0.5      65012              32768     10.0.0.5
10.11.0.4/32  EBgp      10.12.0.5     65012-65022        32768     10.12.0.5
10.11.0.4/32  EBgp      10.12.0.4     65012-65022        32768     10.12.0.4
10.11.0.5/32  EBgp      10.12.0.5     65012-65022        32768     10.12.0.5
10.11.0.5/32  EBgp      10.12.0.4     65012-65022        32768     10.12.0.4
10.11.0.0/16  EBgp      10.12.0.5     65012-65022-65011  32768     10.12.0.5
10.11.0.0/16  EBgp      10.12.0.4     65012-65022-65011  32768     10.12.0.4
10.12.0.4/32  EBgp      10.11.0.5     65011-65022        32768     10.11.0.5
10.12.0.4/32  EBgp      10.11.0.4     65011-65022        32768     10.11.0.4
10.12.0.5/32  EBgp      10.11.0.5     65011-65022        32768     10.11.0.5
10.12.0.5/32  EBgp      10.11.0.4     65011-65022        32768     10.11.0.4
10.12.0.0/16  EBgp      10.11.0.5     65011-65022-65012  32768     10.11.0.5
10.12.0.0/16  EBgp      10.11.0.4     65011-65022-65012  32768     10.11.0.4
```

Let us concentrate on the routes from vnet0 to vnet 21:

```
10.21.0.0/16  EBgp      10.11.0.4     65011-65021        32768     10.11.0.4
10.21.0.0/16  EBgp      10.11.0.5     65011-65021        32768     10.11.0.5
10.21.0.0/16  IBgp      10.0.0.5      65011-65021        32768     10.0.0.5
10.21.0.0/16  EBgp      10.12.0.5     65012-65021        32768     10.12.0.5
10.21.0.0/16  EBgp      10.12.0.4     65012-65021        32768     10.12.0.4
```

How to make vng0 prefer the path over 65011??



## BGP topology: vnet11

This is the most complex table, since vnet11 peers with vnet0, vnet21 and vnet 22

```
C:\>az network vnet-gateway list-bgp-peer-status -g vngbgp -n vng11 -o table
Neighbor    ASN    State      ConnectedDuration    RoutesReceived    MessagesSent    MessagesReceived                                                                                                               ----------  -----  ---------  -------------------  ----------------  --------------  ------------------
10.22.0.5   65022  Connected  00:27:08.6567992     4                 39              39
10.22.0.4   65022  Connected  00:27:07.2673834     4                 38              39
10.21.0.5   65021  Connected  00:32:22.7958145     4                 54              49
10.21.0.4   65021  Connected  00:32:24.8084404     4                 50              47
10.0.0.5    65000  Connected  00:34:48.0548753     4                 53              50
10.0.0.4    65000  Connected  00:34:55.5059938     4                 56              53
10.11.0.4   65011  Unknown                         0                 0               0
10.11.0.5   65011  Connected  01:00:06.8009275     12                92              94
10.22.0.5   65022  Connected  00:00:00.5955963     4                 408             36
10.22.0.4   65022  Connected  00:27:11.5701360     4                 35              37
10.21.0.5   65021  Connected  00:32:22.8168527     4                 50              54
10.21.0.4   65021  Connected  00:00:01.3608463     4                 494             50
10.0.0.5    65000  Connected  00:34:48.0815417     4                 50              53
10.0.0.4    65000  Connected  00:34:58.4974892     4                 52              53
10.11.0.4   65011  Connected  00:58:53.4054202     12                109             94
10.11.0.5   65011  Unknown                         0                 0               0
```

## BGP topology: vnet21

vnet21 peers with vnet11 and vnet 12

```
C:\>az network vnet-gateway list-bgp-peer-status -g vngbgp -n vng21 -o table
Neighbor    ASN    State      ConnectedDuration    RoutesReceived    MessagesSent    MessagesReceived                                                                                                               ----------  -----  ---------  -------------------  ----------------  --------------  ------------------
10.12.0.5   65012  Connected  00:22:35.6581135     10                38              38
10.12.0.4   65012  Connected  00:22:33.0802920     10                36              38
10.11.0.5   65011  Connected  00:30:17.0180933     10                46              49
10.11.0.4   65011  Connected  00:30:19.8657262     10                43              50
10.21.0.4   65021  Unknown                         0                 0               0
10.21.0.5   65021  Connected  00:38:33.2173286     12                242             78
10.12.0.5   65012  Connected  00:22:36.4509213     10                38              38
10.12.0.4   65012  Connected  00:22:36.4509213     10                36              38
10.11.0.5   65011  Connected  00:30:17.8944466     10                49              49
10.11.0.4   65011  Connected  00:30:17.9100807     10                45              54
10.21.0.4   65021  Connected  00:50:15.0916031     12                76              84
10.21.0.5   65021  Unknown                         0                 0               0
```

Let's focus on the routes to vnet0:

```
$ az network vnet-gateway list-learned-routes -g vngbgp -n vng21 -o table | grep '10.0.0.0/16'
10.0.0.0/16   EBgp      10.11.0.4     65011-65000        32768     10.11.0.4
10.0.0.0/16   EBgp      10.11.0.5     65011-65000        32768     10.11.0.5
10.0.0.0/16   IBgp      10.21.0.4     65011-65000        32768     10.21.0.4
10.0.0.0/16   EBgp      10.12.0.5     65012-65000        32768     10.12.0.5
10.0.0.0/16   EBgp      10.12.0.4     65012-65000        32768     10.12.0.4
```

Problem statement, how to make vnet21 prefer the way over vnet11? Please look back at the properties of one of the connections when you created it:

```
az network vpn-connection create -g $rg -n 21to11 --shared-key Microsoft123! --vnet-gateway1 vng21 --vnet-gateway2 vng11 --enable-bgp
{
  "resource": {
    "connectionProtocol": "IKEv2",
    "connectionStatus": "Unknown",
    "connectionType": "Vnet2Vnet",
    "egressBytesTransferred": 0,
    "enableBgp": true,
    "ingressBytesTransferred": 0,
    "provisioningState": "Succeeded",
    "resourceGuid": "...",
    "routingWeight": 10,
    "sharedKey": "...",
    "virtualNetworkGateway1": {
      "id": "/subscriptions/.../resourceGroups/vngbgp/providers/Microsoft.Network/virtualNetworkGateways/vng21",
      "resourceGroup": "vngbgp"
    },
    "virtualNetworkGateway2": {
      "id": "/subscriptions/.../resourceGroups/vngbgp/providers/Microsoft.Network/virtualNetworkGateways/vng11",
      "resourceGroup": "vngbgp"
    }
  }
}
```

As you can see, the only parameter we can modify is the routing weight. Note that this is not exactly the BGP weight, since as you could see in the previous command outputs, all routes are learnt with the default 32768 weight:

```
az network vpn-connection update -g vngbgp -n 21to11 --routing-weight 100
```

## Appendix - VPN Gateway resource

There are not many BGP attributes that can be modified in the VPN gateway itself. As you can see here, it is only the ASN. The peering addresses are automatically assigned out of the GatewaySubnet IP space:

```
$ az network vnet-gateway show -g vngbgp -n vng21
{
  "activeActive": true,
  "bgpSettings": {
    "asn": 65021,
    "bgpPeeringAddress": "10.21.0.4,10.21.0.5",
    "peerWeight": 0
  },
  "customRoutes": null,
  "enableBgp": true,
  "etag": "W/\"...\"",
  "gatewayDefaultSite": null,
  "gatewayType": "Vpn",
  "id": "/subscriptions/.../resourceGroups/vngbgp/providers/Microsoft.Network/virtualNetworkGateways/vng21",
  "ipConfigurations": [
    {
      "etag": "W/\"...\"",
      "id": "/subscriptions/.../resourceGroups/vngbgp/providers/Microsoft.Network/virtualNetworkGateways/vng21/ipConfigurations/vnetGatewayConfig0",
      "name": "vnetGatewayConfig0",
      "privateIpAllocationMethod": "Dynamic",
      "provisioningState": "Succeeded",
      "publicIpAddress": {
        "id": "/subscriptions/.../resourceGroups/vngbgp/providers/Microsoft.Network/publicIPAddresses/pip21a",
        "resourceGroup": "vngbgp"
      },
      "resourceGroup": "vngbgp",
      "subnet": {
        "id": "/subscriptions/.../resourceGroups/vngbgp/providers/Microsoft.Network/virtualNetworks/vnet21/subnets/GatewaySubnet",
        "resourceGroup": "vngbgp"
      },
      "type": "Microsoft.Network/virtualNetworkGateways/ipConfigurations"
    },
    {
      "etag": "W/\"...\"",
      "id": "/subscriptions/.../resourceGroups/vngbgp/providers/Microsoft.Network/virtualNetworkGateways/vng21/ipConfigurations/vnetGatewayConfig1",
      "name": "vnetGatewayConfig1",
      "privateIpAllocationMethod": "Dynamic",
      "provisioningState": "Succeeded",
      "publicIpAddress": {
        "id": "/subscriptions/.../resourceGroups/vngbgp/providers/Microsoft.Network/publicIPAddresses/pip21b",
        "resourceGroup": "vngbgp"
      },
      "resourceGroup": "vngbgp",
      "subnet": {
        "id": "/subscriptions/.../resourceGroups/vngbgp/providers/Microsoft.Network/virtualNetworks/vnet21/subnets/GatewaySubnet",
        "resourceGroup": "vngbgp"
      },
      "type": "Microsoft.Network/virtualNetworkGateways/ipConfigurations"
    }
  ],
  "location": "westeurope",
  "name": "vng21",
  "provisioningState": "Succeeded",
  "resourceGroup": "vngbgp",
  "resourceGuid": "...",
  "sku": {
    "capacity": 2,
    "name": "VpnGw1",
    "tier": "VpnGw1"
  },
  "tags": null,
  "type": "Microsoft.Network/virtualNetworkGateways",
  "vpnClientConfiguration": null,
  "vpnType": "RouteBased"
}
```
