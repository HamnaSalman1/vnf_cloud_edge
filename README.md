# VNF Kubernetes Cloud vs Edge — Kamailio SIP Proxy

COMP5123M Cloud Computing Systems — Coursework 2

## Overview

This project deploys **Kamailio SIP Proxy** as a Virtual Network Function (VNF) across two Kubernetes environments:

- **Cloud**: Minikube on Windows 11 (Docker Desktop)
- **Edge**: K3s inside WSL2 (Ubuntu) on Windows 11

## VNF: Kamailio

Kamailio is an open-source SIP server used in production VoLTE and IMS networks worldwide. It handles SIP REGISTER, INVITE, and BYE messages — the core signalling of every voice call on a 4G/5G network.

**Image used:** `ghcr.io/kamailio/kamailio-ci:latest`

## Environment Setup

### Cloud VM (Minikube — Windows Command Prompt)

```cmd
winget install Kubernetes.minikube
winget install Kubernetes.kubectl
minikube start --driver=docker --cpus=2 --memory=3500mb --disk-size=15g --kubernetes-version=v1.28.0 --profile=cloud-vm
```

### Edge VM (K3s — WSL2/Ubuntu)

```bash
wsl -d Ubuntu
curl -sfL https://get.k3s.io | sh -s - --write-kubeconfig-mode 644 --disable traefik --disable servicelb
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
```

## Deploy VNF

### Cloud (Windows Command Prompt)

```cmd
kubectl config use-context cloud-vm
kubectl apply -f k8s/kamailio-namespace.yaml
kubectl apply -f k8s/kamailio-deployment.yaml
kubectl -n vnf-sip get pods --watch
```

### Edge (WSL2/Ubuntu)

```bash
kubectl apply -f ~/kamailio-namespace.yaml
kubectl apply -f ~/kamailio-deployment.yaml
kubectl -n vnf-sip get pods --watch
```

## Traffic Testing

```cmd
kubectl run iperf-server -n vnf-sip --image=networkstatic/iperf3 --restart=Never -- iperf3 -s
kubectl run iperf-client -n vnf-sip --image=networkstatic/iperf3 --restart=Never -- iperf3 -c <SERVER_IP> -t 30
kubectl run iperf-udp -n vnf-sip --image=networkstatic/iperf3 --restart=Never -- iperf3 -c <SERVER_IP> -u -b 100M -t 30
kubectl run ping-test -n vnf-sip --image=busybox --restart=Never -- ping -c 50 <SERVER_IP>
kubectl top pods -n vnf-sip
```

## Results Summary

| Metric | Cloud (Minikube) | Edge (K3s) |
|--------|-----------------|------------|
| TCP Throughput | 15.3 Gbits/sec | 29.8 Gbits/sec |
| TCP Retransmits | 3,080 | 2,330 |
| UDP Bitrate | 99.7 Mbits/sec | 100 Mbits/sec |
| UDP Jitter | 0.036 ms | 0.022 ms |
| UDP Packet Loss | 0.29% | 0% |
| Ping Avg Latency | 0.193 ms | 0.199 ms |
| Kamailio Memory | 24 MiB | 16 MiB |
| Kamailio CPU | 2m cores | 2m cores |

## Key Finding

The edge environment (K3s/WSL2) outperformed cloud on most metrics due to WSL2's optimised Linux kernel, which avoids the extra Docker-in-Docker virtualisation layer present in Minikube.
