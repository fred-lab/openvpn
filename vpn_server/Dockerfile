FROM alpine:3.8

LABEL maintainer="Fred Gauthier <fred.gauthier.dev@gmail.com>" \
    Description="An Open-VPN container, build with Alpine"

ARG  OPENVPN_VERSION
ENV  OPENVPN_VERSION $OPENVPN_VERSION
ARG  KEY_BITS
ENV  KEY_BITS $KEY_BITS
ARG  SERVER_NAME
ENV  SERVER_NAME $SERVER_NAME
ARG  SERVER_PORT
ENV  SERVER_PORT $SERVER_PORT
ARG  SERVER_PROTOCOL
ENV  SERVER_PROTOCOL $SERVER_PROTOCOL
ARG  PRIMARY_DNS_IP
ENV  PRIMARY_DNS_IP $PRIMARY_DNS_IP
ARG  SERVER_IP
ENV  SERVER_IP $SERVER_IP
ARG  SERVER_USERNAME
ENV  SERVER_USERNAME $SERVER_USERNAME
ARG  SERVER_USERGROUP
ENV  SERVER_USERGROUP $SERVER_USERGROUP

RUN apk update \
    # Install dependencies
    && apk add --no-cache --virtual install-dependencies vim curl tar \
    # Permanent dependencies
    && apk add --no-cache openvpn openssl iptables \
    # Iptables rules
    # && echo $SERVER_IP \
    # && printenv \
    # && iptables -t nat -A POSTROUTING -s 10.8.0.0/24 ! -d 10.8.0.0/24 -j SNAT --to $SERVER_IP \
    # && iptables -I INPUT -p $SERVER_PROTOCOL --dport $SERVER_PORT -j ACCEPT \
    # && iptables -I FORWARD -s 10.8.0.0/24 -j ACCEPT \
    # && iptables -I FORWARD -m state --state RELATED,ESTABLISHED -j ACCEPT \
    # && iptables -t nat -D POSTROUTING -s 10.8.0.0/24 ! -d 10.8.0.0/24 -j SNAT --to $SERVER_IP \
    # && iptables -D INPUT -p $SERVER_PROTOCOL --dport $SERVER_PORT -j ACCEPT \
    # && iptables -D FORWARD -s 10.8.0.0/24 -j ACCEPT \
    # && iptables -D FORWARD -m state --state RELATED,ESTABLISHED -j ACCEPT \
    # Create a default user
    && addgroup $SERVER_USERGROUP \
    && adduser -SHDG $SERVER_USERNAME $SERVER_USERGROUP \
    && cd /etc/openvpn \
    # Edit server conf
    && curl -L "https://raw.githubusercontent.com/OpenVPN/openvpn/master/sample/sample-config-files/server.conf" -o server.conf \
    && sed -ire 's/ca ca.crt/ca cert\/ca.crt/g' server.conf \
    && sed -ire 's/cert server.crt/cert cert\/'"${SERVER_NAME}"'.crt/g' server.conf \
    && sed -ire 's/key server.key/key cert\/'"${SERVER_NAME}"'.key/g' server.conf \
    && sed -ire 's/port 1194/port '"${SERVER_PORT}"'/g' server.conf \
    && sed -ire 's/proto udp/proto '"${SERVER_PROTOCOL}"'/g' server.conf \
    && sed -ire 's/;user nobody/user '"${SERVER_USERNAME}"'/g' server.conf \
    && sed -ire 's/;group nobody/group '"${SERVER_USERGROUP}"'/g' server.conf \
    && sed -ire 's/;push "route 192.168.10.0 255.255.255.0"/push "route '"${SERVER_IP}"' 255.255.255.0"/g' server.conf \
    && sed -ire 's/;log /log /g' server.conf \
    && sed -ire 's/;push "dhcp-option DNS 208.67.222.222"/push "dhcp-option DNS '"${PRIMARY_DNS_IP}"'"/g' server.conf \
    && sed -ie "\$a" server.conf \ 
    && sed -ie "\$a# Use to check if the client authorization is not revoked.\ncrl-verify cert/crl.pem" server.conf \ 
    # Generate a static key
    && openvpn --genkey --secret ta.key \
    && echo "Generate Diffie Hellman key " \
    && openssl dhparam -out /etc/openvpn/dh${KEY_BITS}.pem ${KEY_BITS} \
    && ip addr \
    && route -n \
    # Clean up
    # && apk del --purge install-dependencies \
    && rm -rf /var/cache/apk/*

WORKDIR /etc/openvpn

# USER ${SERVER_USERNAME}

EXPOSE ${SERVER_PORT}/${SERVER_PROTOCOL}

CMD [ "openvpn", "server.conf" ]