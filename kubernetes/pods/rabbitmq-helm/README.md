
<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [RabbitMQ](#rabbitmq)
- [Using with make file:](#using-with-make-file)
  - [Install:](#install)
  - [Updating:](#updating)
  - [Deleting:](#deleting)
  - [Listing helm charts:](#listing-helm-charts)
- [Using Manually:](#using-manually)
- [Updating](#updating)
  - [Enabling SSL Support](#enabling-ssl-support)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

RabbitMQ
============

We are using this helm chart: https://github.com/kubernetes/charts/tree/master/stable/rabbitmq-ha

Using helm version: 2.8.1

# Using with make file:

```
export KUBE_NAMESPACE=rabbitmq
```

## Install:
```
make install
```

## Updating:
```
make upgrade
```

## Deleting:
```
make delete
```

## Listing helm charts:
```
make list
```

# Using Manually:
```
export KUBE_NAMESPACE=rabbitmq
```

```
helm repo add stable https://kubernetes-charts.storage.googleapis.com/

helm install \
--version 1.6.1 \
--name ${KUBE_NAMESPACE}-rabbitmq \
--namespace ${KUBE_NAMESPACE} \
--values ./kubernetes/pods/rabbitmq-helm/values.yaml \
--debug \
stable/rabbitmq-ha
```

# Updating
```
helm upgrade \
--version 1.6.1 \
--values ./kubernetes/pods/rabbitmq-helm/values.yaml \
${KUBE_NAMESPACE}-rabbitmq \
stable/rabbitmq-ha
```

## Enabling SSL Support

RabbitMQ Documentation: https://www.rabbitmq.com/ssl.html#enabling-ssl

Add in the certificates in this section:

```
rabbitmqCert:
  enabled: true

  # Specifies an existing secret to be used for SSL Certs
  existingSecret: ""

  ## Create a new secret using these values
  cacertfile: |
  certfile: |
  keyfile: |
```

The certs has to be base64 encoded:
```
cat certfile.ca | base64 -w0
```

Enable the SSL configurations:
```
rabbitmqAmqpsSupport:
  enabled: true

  # NodePort
  amqpsNodePort: 5671

  # SSL configuration
  config: |
    listeners.ssl.default             = 5671
    ssl_options.cacertfile            = /etc/cert/cacert.pem
    ssl_options.certfile              = /etc/cert/cert.pem
    ssl_options.keyfile               = /etc/cert/key.pem
    ssl_options.verify                = verify_peer
    ssl_options.fail_if_no_peer_cert  = false
```

