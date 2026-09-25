# 1.ipsec.conf

```
config setup
    charondebug="ike 4, knl 2, cfg 3"
conn my-vpn
    keyexchange=ikev2
    type=tunnel

    ike=aes128-sha256-modp1536!
    esp=aes128-sha256!
    fragmentation=yes
    leftsourceip=%config
    mobike=no
    forceencaps=yes

    left=%defaultroute
    leftid=mark

    leftauth=eap
    eap_identity=mark

    right=195.229.66.122
    rightauth=psk

    rightid=%any
    aaa_identity=%any
    rightsubnet=0.0.0.0/0

    dpdaction=clear
    dpddelay=300s

    auto=add
```
# 2./etc/ipsec.secrets 
```
#mark : EAP "Medad@123456"
%any %any : PSK "b(x814ZLPc3XUi1#Q6$sdS#s"
mark : EAP "Medad@123456"
```
# 三层
```
Internet
   │
   │ UDP 500
   │
IKE_SA_INIT
   │
   ↓
IPsec/IKE SA 建立
   │
   │ UDP 4500（NAT-T）
   ↓
┌──────────────────────────┐
│        IPsec / ESP        │
│ ┌──────────────────────┐ │
│ │      L2TP            │ │
│ │   UDP 1701            │ │
│ │ ┌──────────────────┐ │ │
│ │ │       PPP        │ │ │
│ │ │                  │ │ │
│ │ │   EAP / CHAP     │ │ │
│ │ └──────────────────┘ │ │
│ └──────────────────────┘ │
└──────────────────────────┘
```
# 按照xvlan的逻辑。因为可以把l2tp理解成一个二层链路协议。
