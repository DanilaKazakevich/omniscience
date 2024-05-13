
```bash
      containers:
        - name: wireguard
          image: linuxserver/wireguard
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 51820
          env:
            - name: PUID
              value: '1000'
            - name: PGID
              value: '1000'
          securityContext:
            privileged: true
            capabilities:
              add:
                - NET_ADMIN
                - SYS_MODULE
          volumeMounts:
            - name: wireguard-config
              mountPath: /config/wg0.conf
              subPath: wg0.conf
      volumes:
        - name: wireguard-config
          secret:
            secretName: wireguard-secret


# https://keys.saritasa.cloud/cred/detail/k366wsqHjACFEUa5tm2GSA/
apiVersion: v1
kind: Secret
metadata:
  name: wireguard-secret
  namespace: saritasa-crm-slack-app
type: Opaque
stringData:
  wg0.conf: |
    [Interface]
    Address = 10.10.50.4/32
    # PublicKey gjt0ysTJRVg/D586RwCurn0NATwIM10ddpFeV48Zy0Q=
    PrivateKey = xxxxxxxxxxxxxxxx=
    PostUp = iptables -A FORWARD -i wg0 -j ACCEPT
    PostDown = iptables -D FORWARD -i wg0 -j ACCEPT
    ListenPort = 51820
    MTU = 1400

    [Peer]
    # friendly_name = saritasa_cloud_wireguard_ec2
    Endpoint = 44.227.112.181:51820
    PublicKey = /Zc2T5wqVhUBkKM6rrodRKn4k2SG0FT7JAZg5skw7Aw=
    PresharedKey = xxxxxxxxxxxxxxxxxxxxxxxxxx=
    AllowedIPs = 10.10.50.1/32,192.168.0.0/16
    PersistentKeepalive = 25

```