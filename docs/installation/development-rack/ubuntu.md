# Ubuntu

## Initial Setup

### Docker

    $ sudo apt install docker.io

### Kubernetes

    $ snap install microk8s --classic --channel=1.13/stable
    $ microk8s.enable dns storage
    $ mkdir -p ~/.kube
    $ microk8s.config > ~/.kube/config
    $ sudo snap restart microk8s

### Terraform

- Install [Terraform](https://learn.hashicorp.com/terraform/getting-started/install.html)

### Convox CLI

    $ curl -L https://github.com/convox/convox/releases/latest/download/convox-linux -o /tmp/convox
    $ sudo mv /tmp/convox /usr/local/bin/convox
    $ sudo chmod 755 /usr/local/bin/convox

## Installation

Install a local Rack named `dev`.

    $ convox rack install local dev

## DNS Setup

Set `*.convox` to be resolved by the local Rack's DNS server.

    $ sudo mkdir -p /usr/lib/systemd/resolved.conf.d
    $ sudo bash -c "printf '[Resolve]\nDNS=$(kubectl get service/resolver-external -n convox-system -o jsonpath="{.spec.clusterIP}")\nDomains=~convox' > /usr/lib/systemd/resolved.conf.d/convox.conf"
    $ systemctl daemon-reload
    $ systemctl restart systemd-networkd systemd-resolved

## CA Trust

To remove browser warnings about untrusted certificates for local applications
you can trust the Rack's CA certificate.

This certificate is generated on your local machine and is unique to your Rack.

    $ kubectl get secret/ca -n dev-system -o jsonpath="{.data.tls\.crt}" | base64 -d > /tmp/ca
    $ sudo mv /tmp/ca /usr/local/share/ca-certificates/convox.crt
    $ sudo update-ca-certificates
    $ sudo snap restart microk8s
    $ sudo service docker restart