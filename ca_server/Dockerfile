FROM alpine:3.8

LABEL maintainer="Fred Gauthier <fred.gauthier.dev@gmail.com>" \
    Description="An Open-VPN container, build with Alpine"

ARG  OPENVPN_VERSION
ENV  OPENVPN_VERSION $OPENVPN_VERSION
ARG  SERVER_NAME
ENV  SERVER_NAME $SERVER_NAME

ARG  EASYRSA_VERSION
ENV  EASYRSA_VERSION $EASYRSA_VERSION
ARG  EASYRSA_REQ_COUNTRY
ENV  EASYRSA_REQ_COUNTRY $EASYRSA_REQ_COUNTRY
ARG  EASYRSA_REQ_PROVINCE
ENV  EASYRSA_REQ_PROVINCE $EASYRSA_REQ_PROVINCE
ARG  EASYRSA_REQ_CITY
ENV  EASYRSA_REQ_CITY $EASYRSA_REQ_CITY
ARG  EASYRSA_REQ_ORG
ENV  EASYRSA_REQ_ORG $EASYRSA_REQ_ORG
ARG  EASYRSA_REQ_EMAIL
ENV  EASYRSA_REQ_EMAIL $EASYRSA_REQ_EMAIL
ARG  EASYRSA_REQ_OU
ENV  EASYRSA_REQ_OU $EASYRSA_REQ_OU
ARG  EASYRSA_CERT_EXPIRE
ENV  EASYRSA_CERT_EXPIRE $EASYRSA_CERT_EXPIRE
ARG  EASYRSA_CRL_DAYS
ENV  EASYRSA_CRL_DAYS $EASYRSA_CRL_DAYS

ENV EASYRSA_BATCH="yes"

RUN apk update \
    # Install dependencies
    && apk add --no-cache --virtual install-dependencies vim curl tar  \
    # Permanent dependencies
    && apk add --no-cache openvpn openssl \
    && echo "Install Easy RSA" \
    && curl -L "https://github.com/OpenVPN/easy-rsa/releases/download/v${EASYRSA_VERSION}/EasyRSA-unix-v${EASYRSA_VERSION}.tgz" | tar -xzC /usr/share \
    && mv /usr/share/EasyRSA-v${EASYRSA_VERSION} /usr/share/easyrsa \
    && cp -r /usr/share/easyrsa /etc/openvpn \
    && ln -s /usr/share/easyrsa/easyrsa /usr/local/bin \
    && ln -s /usr/share/easyrsa/x509-types /usr/local/bin \
    # Copy EasyRSA Vars file
    && cd /etc/openvpn \
    # && cp ./easyrsa/vars.example ./easyrsa/vars \
    # Initiate the public key infrastructure (PKI)
    && easyrsa init-pki \
    # Build CA : create a CA certificate (pki/ca.crt) and a key (pki/private/ca.key)
    && cp ./easyrsa/openssl-easyrsa.cnf ./pki/ \
    && easyrsa build-ca nopass \
    # Automatically generate server certificates
    && easyrsa build-server-full ${SERVER_NAME} nopass \
    #  Automatically generate client certificates
    && easyrsa build-client-full laptop nopass \
    # Generate a certificate revocation list (CRL). To verify if client certificates are authorized.
    && easyrsa gen-crl \
    #Create a folder to share certificates and others files to the vpn server
    && mkdir -p $SERVER_NAME \
    # Copy server certificates
    && cp ./pki/private/${SERVER_NAME}.key ./${SERVER_NAME}/ \
    && cp ./pki/issued/${SERVER_NAME}.crt ./${SERVER_NAME}/ \
    && cp ./pki/reqs/${SERVER_NAME}.req ./${SERVER_NAME}/ \
    && cp ./pki/crl.pem ./${SERVER_NAME}/ \
    # Make cert/crl.pem readable for non-root user
    && chmod +r ./${SERVER_NAME}/crl.pem \
    && cp ./pki/ca.crt ./${SERVER_NAME}/ \
    #Create a share folder for the clients certificates
    && mkdir clients \
    # Copy clients certificates
    # set a passphrase easyrsa set-rsa-pass servername
    # && easyrsa set-rsa-pass ca toto \
    # Clean up
    # && apk del --purge install-dependencies \
    && rm -rf /var/cache/apk/*

WORKDIR /etc/openvpn

