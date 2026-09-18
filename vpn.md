# vpn 

```
vpn
fortigate ------> strongwan --------> freeradius --------> entra id
```

## fortigate request and configuration 

### 
1. need fortigate client 7.4.3 (above)


## strongwan request and configuration 

###
1. need version 6.0.1(above)
2. configuration
  a. strongwan.conf
    ```
    # strongswan.conf - strongSwan configuration file
    #
    # Refer to the strongswan.conf(5) manpage for details
    #
    # Configuration changes should be made in the included files
    
    charon {
    load_modular = yes
            #load = random nonce openssl pem pkcs1 curl revocation vici kernel-netlink socket-default eap-identity eap-mschapv2 updown
            #load = random nonce openssl pem pkcs1 curl revocation vici kernel-netlink socket-default eap-identity eap-mschapv2 eap-radius updown
            load = random nonce openssl pem pkcs1 curl revocation vici kernel-netlink socket-default eap-identity eap-mschapv2 eap-ttls updown eap-radius 
    
    plugins {
    include strongswan.d/charon/*.conf
    }
    }
    
    include strongswan.d/*.conf
    ```
