```yaml
# cat docker-compose.yml 
version: '3'
services:
 emqx:
   image: emqx/emqx:v4.1.1
   container_name: emqx
   restart: always
   environment:
     - EMQX_LISTENER__TCP__EXTERNAL=1883
     - EMQX_LOADED_PLUGINS=emqx_management,emqx_recon,emqx_retainer,emqx_dashboard
     - EMQX_AUTH__MYSQL__SERVER=xxx:xxx:xxx:xxx:3306   # xxx:xxx:xxx:xxx是IP
     - EMQX_AUTH__MYSQL__USERNAME=root     
     - EMQX_AUTH__MYSQL__PASSWORD=XXXX                 # XXXX是密码    
     - EMQX_AUTH__MYSQL__DATABASE=mqtt
     - EMQX_AUTH__MYSQL__PASSWORD_HASH=sha256
     - EMQX_ALLOW_ANONYMOUS=true
     - EMQX_ACL_NOMAtCH=allow
     - EMQX_MQTT__MAX_PACKET_SIZE=10MB
     - EMQX_NAME=emqx
   network_mode: host
```

