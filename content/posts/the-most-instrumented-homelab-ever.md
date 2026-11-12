+++
title = "The Most Instrumented Homelab Ever?"
date = 2026-11-12T18:01:46.956Z
draft = false
slug = "the-most-instrumented-homelab-ever"
description = "Adventures in observability with Grafana Alloy + Grafana Cloud"
cover.image = "/images/substack/the-most-instrumented-homelab-ever/00-1e537db174.png"
canonicalURL = "https://ezebunandu.substack.com/p/17414636-5111-4434-a249-12be0ce75b59?postPreview=free&updated=2026-08-07T16%3A04%3A25.862Z&audience=everyone&free_preview=false&freemail=true"
originalURL = "https://ezebunandu.substack.com/p/17414636-5111-4434-a249-12be0ce75b59?postPreview=free&updated=2026-08-07T16%3A04%3A25.862Z&audience=everyone&free_preview=false&freemail=true"
tags = ["substack-migration"]
+++

> Adventures in observability with Grafana Alloy + Grafana Cloud

### Extreme o11y

I am on a self-assigned mission to build the most instrumented homelab ever or to die trying. I have kept a homelab for a couple of years now, mostly as a playground to abuse Kubernetes and the Phillips Hue API. It was fun building and evolving on that but I knew it was time to change things up when the VM’s running the k3s cluster started running out of disk space. The experience of expanding the disk volume through the Proxmox UI adhoc felt painful, and my cluster upgrade hygiene left a lot to be desired. For the revamp, I set the goal of having every piece of the infrastructure deployed as code and version controlled. I have a collection of interesting stories on some of the design choices I and my coding agents have made so far maybe worth sharing. For today though, I want to tell you about my adventures instrumenting the heck out of the new environment. First off, a quick introduction to the refreshed hardware and setup.

### A Box of Goodies

The new homelab, aka labbie 2.0, is a Minisforum MS-A2 and two Lenovo 910q joined in a Proxmox cluster. The entire ensemble is housed inside of a KWS-1 modular 10inch rack that I 3D printed and absolutely love. There are 6 Talos Linux nodes (3 control plane and 3 worker nodes) running an assortment of workloads, including Hashicorp Vault for secrets management, Harbor for container registry, and ArgoCD for continuous deployment. Labbie 1.0 had built it’s observability layer on the k3s cluster itself using a self-hosted Grafana (LGTM) stack, installed as a Helm chart. Rather than a self-hosted stack this time, I decided to use a collector to ship telemetry to Grafana cloud. This would somewhat help with isolating telemetry storage and analysis from cluster failure issues, though at the trade-off of depending on my LAN’s internet connectivity being up.

Having decided to use the Grafana Cloud stack for telemetry storage and analysis, the next big architectural decision I needed to make was how deep inside the infrastructure to send instrumentation probes. Labbie 1.0 had only been instrumented at the level of the kubernetes cluster using the aforementioned Helm chart. This was a choice born chiefly out of a delicate mix of laziness and pragmatism, but left me blind to operational issues lower down the stack, such as the VM nodes silently running out of disk space. This time around, I would also instrument the Proxmox nodes. Heaven forbid us ever needing to expand disk volumes in a rush through the Proxmox UI again.

![](/images/substack/the-most-instrumented-homelab-ever/01-4a8d35ac95.png)

My New HomeLab hardware in a beautiful KWS-2 10 inch modular rack

### Grafana Alloy; A Collector’s Collector

As someone who works at Grafana Labs, I am certainly not without bias, but I genuinely enjoy working with Grafana’s products both professionally and personally, and so there was only one natural choice of telemetry collector for my homelab, the open-source, OpenTelemetry Collector distribution, Grafana Alloy.

Alloy would enable us to collect telemetry data from the nodes, to transform and process the data as may be required, and then to send it on to Grafana Cloud remote storage backends. In our setup, we will use Grafana cloud versions of Loki and Mimir as storage backends for our logs and metrics. Any collection and transformation required will be done by Alloy. I suppose it does help that Alloy’s expression-based configuration language would enable us to handle transformations big and small. Because this is just a homelab and we are trying to optimize for the amount of fun had per prompt or keystroke, we will start simple and add things as we go along, driven by an the observability roadmap that has been handed down by the Homelab technical leadership team

### Architecting For Observability

I considered a few possible ways to set up the collector(s). One option would have been to use a Sidecar VM on each host or one shared monitoring VM to collect from the Proxmox hosts and Talos cluster, to avoid touching the hypervisor’s own root namespace. That approach would have made collecting Kubernetes node-level host metrics, pod logs and kube-apiserver audit logs somewhat tricky.

![](/images/substack/the-most-instrumented-homelab-ever/02-7ab8e75a7a.png)

The approach I instead settled on was to use a native agent on each host machine. This allows access to signals that only exist locally while enabling us to use PVE 9’s native OTLP push to collect metrics at the Proxmox cluster layer. In this way, we get access to both native Linux host metrics and Proxmox-layer telemetry, but at the cost of running with root privileges on the hypervisor.

![](/images/substack/the-most-instrumented-homelab-ever/03-f481be3a36.png)

### Instrumenting the Proxmox Nodes; The Gory Details

We are going to run one instance of Alloy as a systemd unit on each physical host running the Proxmox hypervisor. Since Proxmox 9.x, Proxmox nodes can now also expose an OTEL push server that can send internal telemetry out via OTLP/HTTP, enabling collection of per-guest, per-storage-pool and cluster-level metrics. The combination of the host-level metrics and pvestatd metrics give us a complete picture of things at the Proxmox layer, so that we can answer questions about how healthy each box is, but also how healthy is what’s running on it.

The Alloy instances and their config are deployed with Terraform over ssh. The main Terraform module creates a Grafana cloud access policy for writing metrics and logs and a policy\_token, enabling each instance to write to Grafana cloud. The Cloud credentials are written as an env file that the systemd unit references via EnvironmentFile, so that values do not need to be included in the config source code.

```
  # Install Alloy binary if absent or version differs, then start the service.
  provisioner "remote-exec" {
    inline = [
      "set -euo pipefail",
      "# Install binary if version differs",
      "INSTALLED=$(alloy --version 2>/dev/null | awk '{print $3}' || echo none)",
      "TARGET=v${var.alloy_version}",
      "if [ \"$INSTALLED\" != \"$TARGET\" ]; then",
      "  echo \"Installing Alloy $TARGET (installed: $INSTALLED)\"",
      "  # Proxmox ships without unzip; install it (tolerate a noisy apt-get update on no-subscription hosts).",
      "  command -v unzip >/dev/null 2>&1 || { apt-get update -qq || true; DEBIAN_FRONTEND=noninteractive apt-get install -y -qq unzip; }",
      "  curl -fsSL https://github.com/grafana/alloy/releases/download/$TARGET/alloy-linux-amd64.zip -o /tmp/alloy.zip",
      "  cd /tmp && unzip -o alloy.zip alloy-linux-amd64",
      "  install -m 755 /tmp/alloy-linux-amd64 /usr/local/bin/alloy",
      "  rm -f /tmp/alloy.zip /tmp/alloy-linux-amd64",
      "fi",
      "systemctl daemon-reload",
      "systemctl enable alloy",
      "systemctl restart alloy",
      # Type=simple returns success at exec; settle, then verify it's truly up
      # (a config error crash-loops, which a bare is-active would miss) and that
      # the OTLP receiver is bound before the metric-server step tests it.
      "sleep 3",
      "systemctl is-active --quiet alloy || { journalctl -u alloy --no-pager -n 30; exit 1; }",
    ]
  }
```

Each Alloy instance is configured using a template file.

```
// ── Proxmox VE guest / cluster / storage metrics (PVE 9 native OTLP) ─────────
// pvestatd on this host pushes node, guest, and storage metrics over OTLP/HTTP
// to the local receiver below (configured cluster-wide to target localhost, so
// each node ships its own slice). We stamp identity labels to match the
// node_exporter scrape, then convert OTLP → Prometheus before remote_write.
// node_exporter (above) still owns deep host-OS metrics; this adds the
// virtualization layer (per-guest, storage pools, cluster) it can't see.
otelcol.receiver.otlp "pve" {
  http {
    endpoint = "127.0.0.1:4318"
  }

  output {
    metrics = [otelcol.processor.attributes.pve.input]
  }
}

otelcol.processor.attributes "pve" {
  action {
    key    = "instance"
    value  = "${hostname}"
    action = "upsert"
  }
  action {
    key    = "cluster"
    value  = "devops-cluster"
    action = "upsert"
  }

  output {
    metrics = [otelcol.exporter.prometheus.pve.input]
  }
}
```

### The Easier Button: Grafana K8S Monitoring Helm

![](/images/substack/the-most-instrumented-homelab-ever/04-c7cc97f9a7.png)

### Putting It All Together; Operational Monitoring with Grafana Alerting + Discord
