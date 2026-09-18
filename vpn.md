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
  b. swanctl.conf
     ```
       pools {
          vpn-pool {
              addrs = 10.250.0.0/24
          }
      }
      connections {
          forticlient-ipsec {
              version = 2
              #aggressive = yes
      
              local_port = 500
      #        proposals = aes128-sha1-modp1024,default
      #proposals = aes128-sha1-modp1024
      proposals = aes256-sha1-modp1024
      
              local {
                  auth = psk
                  #id = server.strongswan.org
          #certs = serverCert.pem
              }
      remote {
                  #auth = eap-mschapv2
                  # correct one auth = psk
                  #auth = eap
                  auth = eap-radius
          id = %any 
          #eap_id = %any
                  #auth = xauth-generic
              }
              pools = vpn-pool
              children {
                  net {
                      local_ts = 10.0.0.0/8
      #updown = /usr/local/libexec/ipsec/_updown iptables
              remote_ts = dynamic
              esp_proposals = aes256-sha1,aes128-sha1
                      #esp_proposals = aes128-sha1-modp1024,aes128-sha1
                  }
              }
          }
      }
      
      secrets {
          ike-psk {
      id = %any
              secret = "FortiClientPsk123!"
          }
          #eap-user {
          #    id = testuser
          #    secret = "TestPassword123"
          #}
      }
      include conf.d/*.conf
     ```
   c. charon-logging.conf 
     ```
        filelog {
        strongswan {
            path = /var/log/strongswan.log

            # 普通日志
            default = 4

            # IKE 协商
            ike = 2

            # 配置相关
            cfg = 2

            # 可选：立即刷新日志
            flush_line = yes

            # 可选：日志追加到现有文件
            append = yes
        }
     ```
   d. strongswan.d/charon/eap-radius.conf 
     ```
     
      eap-radius {
          load = yes
          xauth {
              # 将 XAuth 身份凭据直接打成 RADIUS Access-Request 发送
          }
          servers {
              freeradius_local {
                  address = 127.0.0.1
                  secret = ShareYourSecretWithStrongswan
                  #port = 1812
                  auth_port = 1812
              }
          }
      }

     ```
## freeradius request and configuration 
见链接
###
1. version FreeRADIUS Version 3.2.11 (above)
2. client.conf
   ```
   client strongswan_gateway {
    ipaddr = 127.0.0.1 # if on same server, or change to strongSwan's LAN IP
    secret = "ShareYourSecretWithStrongswan"
    }
   ```
3. /usr/local/radius/etc/raddb/mods-enabled/eap
   ``` add or update ---new 
   tls-config tls-common {
      private_key_file = /etc/letsencrypt/live/vgate.app.ayshei.com/privkey.pem
      certificate_file = /etc/letsencrypt/live/vgate.app.ayshei.com/fullchain.pem
      ca_file = /etc/letsencrypt/live/vgate.app.ayshei.com/chain.pem
   ```
#### for the certification we need to sign 
4. /usr/local/radius/etc/raddb/mods-config/files/authorize
   ``` just for test local user ,add these under the file 
    
    testuser    Cleartext-Password := "TestPassword123"
    Nithins-MacBook-Pro.local    Cleartext-Password := "TestPassword1231234"
    ```
5. /usr/local/radius/etc/raddb/sites-enabled/default
   ``` add into  authorize section
      if (&MS-CHAP-User-Name) {
        update request {
            Tmp-String-0 := &MS-CHAP-User-Name
        }
            linelog
        }
   ```
   ``` add into anthenticate

   ```
6. /usr/local/radius/etc/raddb/mods-available/files
7. /usr/local/radius/etc/raddb/sites-enabled/inner-tunnel
   ``` add into authorize
        if (&User-Password) {
        update control {
            Auth-Type := PAP
        }
    }
   
   ```
   ``` add into authorize before pap
        rest 
   
   ```
   ``` add into authenticate

       Auth-Type PAP {
            rest
        }
  ```

8.  mods-enabled/rest
  ```
  rest {
    connect {
        uri = "https://login.microsoftonline.com"
        tls {
            # Standard web verification
            require_cert = "allow"
        }
    }
    
    authenticate {
        # Construct the Microsoft OAuth2 password check request
        uri = "${..connect.uri}/<tenant id>/oauth2/v2.0/token"
        method = 'post'
body = 'post'
        data = 'grant_type=password&client_id=<application id>&client_secret=<secret>&scope=user.read&username=%{User-Name}&pass
word=%{User-Password}'
        
        # If Microsoft returns a 200 OK (token generated), the password is correct!
        valid_codes = 200
    }
  }
  ```

## server  request and configuration
1. kernel forward
  ```
  sysctl -w net.ipv4.ip_forward=1
  ```
2. iptables 转发网络
  ```
  iptables -t nat -A POSTROUTING \
    -s 10.250.0.0/24 \
    -d 192.168.100.0/24 \
    -j MASQUERADE

  iptables -A FORWARD \
    -s 10.250.0.0/24 \
    -d 192.168.100.0/24 \
    -j ACCEPT

  iptables -A FORWARD \
      -s 192.168.100.0/24 \
      -d 10.250.0.0/24 \
      -m conntrack --ctstate ESTABLISHED,RELATED \
      -j ACCEPT
  ```
