# Getting started with Vault - a developer tutorial

## Introduction

This guide will outline how to get started with Hashicorp Vault from a application developer / user perspective. We will be using k3s with the Vault chart in dev mode to rapidly get Vault working and begin experimentation, but a production instance of Vault should not be used in dev mode. Though your authentication method may be different, your user experience should be consistent.

K3s only works on linux distributions, however there are other options for Windows users, such as Minikube. After installing one of these options, the rest of the Vault installation process will be the same.

For a more in-depth look at Vault on Kubernetes, take a look at the [official vault on kubernetes docs](https://learn.hashicorp.com/collections/vault/kubernetes). They have some great and in depth documentation. This repo was created to summarize my process of getting this up and running as quickly and simply as possible.

## Table of Contents

### Using Vault

1. [Getting k3s](k3s_install.md)
2. [Installing Vault in dev mode](vault_install.md)
3. [Secrets Engines](secrets_engines.md)
4. [Create a KV secret with the kv_v2 engine](kv_secret/intro.md)
    - [Web UI](kv_secret/web_ui.md)
    - [Command line](kv_secret/vault_cli.md)
    - [API](kv_secret/vault_api.md)
5. [Injecting secrets with Vault Helm Sidecar](inject_sidecar.md)

### Integrating Vault with Jenkins

1. [Installing a local Jenkins instance](jenkins_install.md)
2. [Using Vault in Jenkins](vault_in_jenkins.md)