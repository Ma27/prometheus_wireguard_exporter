# Prometheus WireGuard Exporter

[![legal](https://img.shields.io/github/license/mindflavor/prometheus_wireguard_exporter.svg)](LICENSE) ![stability-stable](https://img.shields.io/badge/stability-stable-green.svg)

[![Crate](https://img.shields.io/crates/v/prometheus_wireguard_exporter.svg)](https://crates.io/crates/prometheus_wireguard_exporter) [![cratedown](https://img.shields.io/crates/d/prometheus_wireguard_exporter.svg)](https://crates.io/crates/prometheus_wireguard_exporter) [![cratelastdown](https://img.shields.io/crates/dv/prometheus_wireguard_exporter.svg)](https://crates.io/crates/prometheus_wireguard_exporter)

[![release](https://img.shields.io/github/release/MindFlavor/prometheus_wireguard_exporter.svg)](https://github.com/MindFlavor/prometheus_wireguard_exporter/tree/3.2.2)
[![tag](https://img.shields.io/github/tag/mindflavor/prometheus_wireguard_exporter.svg)](https://github.com/MindFlavor/prometheus_wireguard_exporter/tree/3.2.2)

[![Build Status](https://travis-ci.org/MindFlavor/prometheus_wireguard_exporter.svg?branch=master)](https://travis-ci.org/MindFlavor/prometheus_wireguard_exporter)
[![commitssince](https://img.shields.io/github/commits-since/mindflavor/prometheus_wireguard_exporter/3.2.2.svg)](https://img.shields.io/github/commits-since/mindflavor/prometheus_wireguard_exporter/3.2.2.svg)

## Intro

A Prometheus exporter for [WireGuard](https://www.wireguard.com), written in Rust. This tool exports the `wg show all dump` (or `wg show <interface> dump` if you specify a config file) results in a format that [Prometheus](https://prometheus.io/) can understand. The exporter is very light on your server resources, both in terms of memory and CPU usage.

Starting from release [2.0.2](https://github.com/MindFlavor/prometheus_wireguard_exporter/releases/tag/2.0.2) this exporter supports IPv6 addressess too (thanks to [Maximilian Bosch](https://github.com/Ma27)'s PR [#5](https://github.com/MindFlavor/prometheus_wireguard_exporter/pull/5)).
From release [3.0.0](https://github.com/MindFlavor/prometheus_wireguard_exporter/releases/tag/3.0.0) the exporter allows two label modes: one is to dump every allowed ip in a single label (called `allowed_ips`) along with their subnets. The second one is to create a pair of labels for each allowed ip/subnet pair (called `allowed_ip_0`/`allowed_subnet_0`, `allowed_ip_1`/`allowed_subnet_1` and so on for every allowed ip). The default if the single label mode but you can enable the second mode by specifying the `-s` switch at startup. Thank you [Toon Schoenmakers](https://github.com/schoentoon) for this solution (see [https://github.com/MindFlavor/prometheus_wireguard_exporter/issues/8](https://github.com/MindFlavor/prometheus_wireguard_exporter/issues/8)).

![](extra/01.png)

## Prerequisites

* You need [Rust](https://www.rust-lang.org/) to compile this code. Simply follow the instructions on Rust's website to install the toolchain. If you get weird errors while compiling please try and update your Rust version first (I have developed it on `rustc 1.35.0-nightly (8159f389f 2019-04-06)`).
* You need [WireGuard](https://www.wireguard.com) *and* the `wg` CLI in the path. The tool will call `wg show all dump` and of course will fail if the `wg` executable is not found. If you want I can add the option of specifying the `wg` path in the command line, just open an issue for it.

## Compilation

To compile the latest master version:

```bash
git clone https://github.com/MindFlavor/prometheus_wireguard_exporter.git
cd prometheus_wireguard_exporter
cargo install --path .
```

If you want the latest release you can simply use:

```bash
cargo install prometheus_wireguard_exporter
```

## Usage

Start the binary with `-h` to get the complete syntax. The parameters are:

| Parameter | Mandatory | Valid values | Default | Description |
| -- | -- | -- | -- | -- |
| `-v` | no | <switch> | | Enable verbose mode.
| `-p` | no | any valid ip address | 0.0.0.0 | Specify the service address. This is the address your Prometheus instance should point to.
| `-p` | no | any valid port number | 9586 | Specify the service port. This is the port your Prometheus instance should point to.
| `-n` | no | path to the wireguard configuration file | | This flag adds the *friendly_name* attribute to the exported entries. See [Friendly names](#friendly-names) for more details.
| `-s` | no | <switch> | off | Enable the allowed ip + subnet split mode for the labels.
| `-r` | no | <switch> | off | Exports peer's remote ip and port as labels (if available).

Once started, the tool will listen on the specified port (or the default one, 9586, if not specified) and return a Prometheus valid response at the url `/metrics`. So to check if the tool is working properly simply browse the `http://localhost:9586/metrics` (or whichever port you choose).

## Friendly Names

Starting from version 1.2 you can instruct the exporter to append a *friendly name* to the exported entries. This can make the output more understandable than using the public keys. For example this is the standard output:

```
# HELP wireguard_sent_bytes_total Bytes sent to the peer
# TYPE wireguard_sent_bytes_total counter
wireguard_sent_bytes_total{interface="wg0",public_key="2S7mA0vEMethCNQrJpJKE81/JmhgtB+tHHLYQhgM6kk=",allowed_ips="10.70.0.2/32,10.70.0.66/32"} 3208804
wireguard_sent_bytes_total{interface="wg0",public_key="qnoxQoQI8KKMupLnSSureORV0wMmH7JryZNsmGVISzU=",allowed_ips="10.70.0.3/32"} 0
wireguard_sent_bytes_total{interface="wg0",public_key="L2UoJZN7RmEKsMmqaJgKG0m1S2Zs2wd2ptAf+kb3008=",allowed_ips="10.70.0.4/32"} 0
wireguard_sent_bytes_total{interface="wg0",public_key="MdVOIPKt9K2MPj/sO2NlWQbOnFJ6L/qX80mmhQwsUlA=",allowed_ips="10.70.0.50/32"} 0
wireguard_sent_bytes_total{interface="wg0",public_key="lqYcojJMsIZXMUw1heAFbQHBoKjCEaeo7M1WXDh/KWc=",allowed_ips="10.70.0.40/32"} 0
wireguard_sent_bytes_total{interface="wg0",public_key="928vO9Lf4+Mo84cWu4k1oRyzf0AR7FTGoPKHGoTMSHk=",allowed_ips="10.70.0.80/32"} 0
wireguard_sent_bytes_total{interface="wg0",public_key="wTjv6hS6fKfNK+SzOLo7O6BQjEb6AD1TN9GjwZ08IwA=",allowed_ips="10.70.0.5/32"} 0
# HELP wireguard_received_bytes_total Bytes received from the peer
# TYPE wireguard_received_bytes_total counter
wireguard_received_bytes_total{interface="wg0",public_key="2S7mA0vEMethCNQrJpJKE81/JmhgtB+tHHLYQhgM6kk=",allowed_ips="10.70.0.2/32,10.70.0.66/32"} 71420072
wireguard_received_bytes_total{interface="wg0",public_key="qnoxQoQI8KKMupLnSSureORV0wMmH7JryZNsmGVISzU=",allowed_ips="10.70.0.3/32"} 0
wireguard_received_bytes_total{interface="wg0",public_key="L2UoJZN7RmEKsMmqaJgKG0m1S2Zs2wd2ptAf+kb3008=",allowed_ips="10.70.0.4/32"} 0
wireguard_received_bytes_total{interface="wg0",public_key="MdVOIPKt9K2MPj/sO2NlWQbOnFJ6L/qX80mmhQwsUlA=",allowed_ips="10.70.0.50/32"} 0
wireguard_received_bytes_total{interface="wg0",public_key="lqYcojJMsIZXMUw1heAFbQHBoKjCEaeo7M1WXDh/KWc=",allowed_ips="10.70.0.40/32"} 0
wireguard_received_bytes_total{interface="wg0",public_key="928vO9Lf4+Mo84cWu4k1oRyzf0AR7FTGoPKHGoTMSHk=",allowed_ips="10.70.0.80/32"} 0
wireguard_received_bytes_total{interface="wg0",public_key="wTjv6hS6fKfNK+SzOLo7O6BQjEb6AD1TN9GjwZ08IwA=",allowed_ips="10.70.0.5/32"} 0
# HELP wireguard_latest_handshake_seconds Seconds from the last handshake
# TYPE wireguard_latest_handshake_seconds gauge
wireguard_latest_handshake_seconds{interface="wg0",public_key="2S7mA0vEMethCNQrJpJKE81/JmhgtB+tHHLYQhgM6kk=",allowed_ips="10.70.0.2/32,10.70.0.66/32"} 1562834127
wireguard_latest_handshake_seconds{interface="wg0",public_key="qnoxQoQI8KKMupLnSSureORV0wMmH7JryZNsmGVISzU=",allowed_ips="10.70.0.3/32"} 0
wireguard_latest_handshake_seconds{interface="wg0",public_key="L2UoJZN7RmEKsMmqaJgKG0m1S2Zs2wd2ptAf+kb3008=",allowed_ips="10.70.0.4/32"} 0
wireguard_latest_handshake_seconds{interface="wg0",public_key="MdVOIPKt9K2MPj/sO2NlWQbOnFJ6L/qX80mmhQwsUlA=",allowed_ips="10.70.0.50/32"} 0
wireguard_latest_handshake_seconds{interface="wg0",public_key="lqYcojJMsIZXMUw1heAFbQHBoKjCEaeo7M1WXDh/KWc=",allowed_ips="10.70.0.40/32"} 0
wireguard_latest_handshake_seconds{interface="wg0",public_key="928vO9Lf4+Mo84cWu4k1oRyzf0AR7FTGoPKHGoTMSHk=",allowed_ips="10.70.0.80/32"} 0
wireguard_latest_handshake_seconds{interface="wg0",public_key="wTjv6hS6fKfNK+SzOLo7O6BQjEb6AD1TN9GjwZ08IwA=",allowed_ips="10.70.0.5/32"} 0
```

And this is the one augmented with friendly names:

```
# HELP wireguard_sent_bytes_total Bytes sent to the peer
# TYPE wireguard_sent_bytes_total counter
wireguard_sent_bytes_total{interface="wg0",public_key="2S7mA0vEMethCNQrJpJKE81/JmhgtB+tHHLYQhgM6kk=",allowed_ips="10.70.0.2/32,10.70.0.66/32",friendly_name="OnePlus 6T"} 3208804
wireguard_sent_bytes_total{interface="wg0",public_key="qnoxQoQI8KKMupLnSSureORV0wMmH7JryZNsmGVISzU=",allowed_ips="10.70.0.3/32",friendly_name="varch.local (laptop)"} 0
wireguard_sent_bytes_total{interface="wg0",public_key="L2UoJZN7RmEKsMmqaJgKG0m1S2Zs2wd2ptAf+kb3008=",allowed_ips="10.70.0.4/32",friendly_name="cantarch"} 0
wireguard_sent_bytes_total{interface="wg0",public_key="MdVOIPKt9K2MPj/sO2NlWQbOnFJ6L/qX80mmhQwsUlA=",allowed_ips="10.70.0.50/32",friendly_name="frcognoarch"} 0
wireguard_sent_bytes_total{interface="wg0",public_key="lqYcojJMsIZXMUw1heAFbQHBoKjCEaeo7M1WXDh/KWc=",allowed_ips="10.70.0.40/32",friendly_name="frcognowin10"} 0
wireguard_sent_bytes_total{interface="wg0",public_key="928vO9Lf4+Mo84cWu4k1oRyzf0AR7FTGoPKHGoTMSHk=",allowed_ips="10.70.0.80/32",friendly_name="OnePlus 5T"} 0
wireguard_sent_bytes_total{interface="wg0",public_key="wTjv6hS6fKfNK+SzOLo7O6BQjEb6AD1TN9GjwZ08IwA=",allowed_ips="10.70.0.5/32",friendly_name="folioarch"} 0
# HELP wireguard_received_bytes_total Bytes received from the peer
# TYPE wireguard_received_bytes_total counter
wireguard_received_bytes_total{interface="wg0",public_key="2S7mA0vEMethCNQrJpJKE81/JmhgtB+tHHLYQhgM6kk=",allowed_ips="10.70.0.2/32,10.70.0.66/32",friendly_name="OnePlus 6T"} 71420072
wireguard_received_bytes_total{interface="wg0",public_key="qnoxQoQI8KKMupLnSSureORV0wMmH7JryZNsmGVISzU=",allowed_ips="10.70.0.3/32",friendly_name="varch.local (laptop)"} 0
wireguard_received_bytes_total{interface="wg0",public_key="L2UoJZN7RmEKsMmqaJgKG0m1S2Zs2wd2ptAf+kb3008=",allowed_ips="10.70.0.4/32",friendly_name="cantarch"} 0
wireguard_received_bytes_total{interface="wg0",public_key="MdVOIPKt9K2MPj/sO2NlWQbOnFJ6L/qX80mmhQwsUlA=",allowed_ips="10.70.0.50/32",friendly_name="frcognoarch"} 0
wireguard_received_bytes_total{interface="wg0",public_key="lqYcojJMsIZXMUw1heAFbQHBoKjCEaeo7M1WXDh/KWc=",allowed_ips="10.70.0.40/32",friendly_name="frcognowin10"} 0
wireguard_received_bytes_total{interface="wg0",public_key="928vO9Lf4+Mo84cWu4k1oRyzf0AR7FTGoPKHGoTMSHk=",allowed_ips="10.70.0.80/32",friendly_name="OnePlus 5T"} 0
wireguard_received_bytes_total{interface="wg0",public_key="wTjv6hS6fKfNK+SzOLo7O6BQjEb6AD1TN9GjwZ08IwA=",allowed_ips="10.70.0.5/32",friendly_name="folioarch"} 0
# HELP wireguard_latest_handshake_seconds Seconds from the last handshake
# TYPE wireguard_latest_handshake_seconds gauge
wireguard_latest_handshake_seconds{interface="wg0",public_key="2S7mA0vEMethCNQrJpJKE81/JmhgtB+tHHLYQhgM6kk=",allowed_ips="10.70.0.2/32,10.70.0.66/32",friendly_name="OnePlus 6T"} 1562834127
wireguard_latest_handshake_seconds{interface="wg0",public_key="qnoxQoQI8KKMupLnSSureORV0wMmH7JryZNsmGVISzU=",allowed_ips="10.70.0.3/32",friendly_name="varch.local (laptop)"} 0
wireguard_latest_handshake_seconds{interface="wg0",public_key="L2UoJZN7RmEKsMmqaJgKG0m1S2Zs2wd2ptAf+kb3008=",allowed_ips="10.70.0.4/32",friendly_name="cantarch"} 0
wireguard_latest_handshake_seconds{interface="wg0",public_key="MdVOIPKt9K2MPj/sO2NlWQbOnFJ6L/qX80mmhQwsUlA=",allowed_ips="10.70.0.50/32",friendly_name="frcognoarch"} 0
wireguard_latest_handshake_seconds{interface="wg0",public_key="lqYcojJMsIZXMUw1heAFbQHBoKjCEaeo7M1WXDh/KWc=",allowed_ips="10.70.0.40/32",friendly_name="frcognowin10"} 0
wireguard_latest_handshake_seconds{interface="wg0",public_key="928vO9Lf4+Mo84cWu4k1oRyzf0AR7FTGoPKHGoTMSHk=",allowed_ips="10.70.0.80/32",friendly_name="OnePlus 5T"} 0
wireguard_latest_handshake_seconds{interface="wg0",public_key="wTjv6hS6fKfNK+SzOLo7O6BQjEb6AD1TN9GjwZ08IwA=",allowed_ips="10.70.0.5/32",friendly_name="folioarch"} 0
```

In order for this to work, you need to add comments to your wireguard configuration file (below the `[Peer]` definition). The comment will be interpreted as `friendly_name` and added to the entry exported to Prometheus. Note that this is not a standard but, since it's a comment, will not interfere with WireGuard in any way. For example this is how you edit your WireGuard configuration file:

```
[Peer]
PublicKey = lqYcojJMsIZXMUw1heAFbQHBoKjCEaeo7M1WXDh/KWc=
AllowedIPs = 10.70.0.40/32

[Peer]
PublicKey = 928vO9Lf4+Mo84cWu4k1oRyzf0AR7FTGoPKHGoTMSHk=
AllowedIPs = 10.70.0.80/32
```

```
[Peer]
# frcognowin10
PublicKey = lqYcojJMsIZXMUw1heAFbQHBoKjCEaeo7M1WXDh/KWc=
AllowedIPs = 10.70.0.40/32

[Peer]
# OnePlus 5T
PublicKey = 928vO9Lf4+Mo84cWu4k1oRyzf0AR7FTGoPKHGoTMSHk=
AllowedIPs = 10.70.0.80/32
```

As you can see, all you need to do is to add the friendly name as comment (and enable the flag since this feature is opt-in).

This is a sample of the label split mode:

```
# HELP wireguard_sent_bytes_total Bytes sent to the peer
# TYPE wireguard_sent_bytes_total counter
wireguard_sent_bytes_total{interface="wg0",public_key="2S7mA0vEMethCNQrJpJKE81/JmhgtB+tHHLYQhgM6kk=",allowed_ip_0="10.70.0.2",allowed_subnet_0="32",allowed_ip_1="10.70.0.66",allowed_subnet_1="32",friendly_name="OnePlus 6T"} 3208804
wireguard_sent_bytes_total{interface="wg0",public_key="qnoxQoQI8KKMupLnSSureORV0wMmH7JryZNsmGVISzU=",allowed_ip_0="10.70.0.3",allowed_subnet_0="32",friendly_name="varch.local (laptop)"} 0
wireguard_sent_bytes_total{interface="wg0",public_key="L2UoJZN7RmEKsMmqaJgKG0m1S2Zs2wd2ptAf+kb3008=",allowed_ip_0="10.70.0.4",allowed_subnet_0="32",friendly_name="cantarch"} 0
wireguard_sent_bytes_total{interface="wg0",public_key="MdVOIPKt9K2MPj/sO2NlWQbOnFJ6L/qX80mmhQwsUlA=",allowed_ip_0="10.70.0.50",allowed_subnet_0="32",friendly_name="frcognoarch"} 0
wireguard_sent_bytes_total{interface="wg0",public_key="lqYcojJMsIZXMUw1heAFbQHBoKjCEaeo7M1WXDh/KWc=",allowed_ip_0="10.70.0.40",allowed_subnet_0="32",friendly_name="frcognowin10"} 0
wireguard_sent_bytes_total{interface="wg0",public_key="928vO9Lf4+Mo84cWu4k1oRyzf0AR7FTGoPKHGoTMSHk=",allowed_ip_0="10.70.0.80",allowed_subnet_0="32",friendly_name="OnePlus 5T"} 0
wireguard_sent_bytes_total{interface="wg0",public_key="wTjv6hS6fKfNK+SzOLo7O6BQjEb6AD1TN9GjwZ08IwA=",allowed_ip_0="10.70.0.5",allowed_subnet_0="32",friendly_name="folioarch"} 0
# HELP wireguard_received_bytes_total Bytes received from the peer
# TYPE wireguard_received_bytes_total counter
wireguard_received_bytes_total{interface="wg0",public_key="2S7mA0vEMethCNQrJpJKE81/JmhgtB+tHHLYQhgM6kk=",allowed_ip_0="10.70.0.2",allowed_subnet_0="32",allowed_ip_1="10.70.0.66",allowed_subnet_1="32",friendly_name="OnePlus 6T"} 71420072
wireguard_received_bytes_total{interface="wg0",public_key="qnoxQoQI8KKMupLnSSureORV0wMmH7JryZNsmGVISzU=",allowed_ip_0="10.70.0.3",allowed_subnet_0="32",friendly_name="varch.local (laptop)"} 0
wireguard_received_bytes_total{interface="wg0",public_key="L2UoJZN7RmEKsMmqaJgKG0m1S2Zs2wd2ptAf+kb3008=",allowed_ip_0="10.70.0.4",allowed_subnet_0="32",friendly_name="cantarch"} 0
wireguard_received_bytes_total{interface="wg0",public_key="MdVOIPKt9K2MPj/sO2NlWQbOnFJ6L/qX80mmhQwsUlA=",allowed_ip_0="10.70.0.50",allowed_subnet_0="32",friendly_name="frcognoarch"} 0
wireguard_received_bytes_total{interface="wg0",public_key="lqYcojJMsIZXMUw1heAFbQHBoKjCEaeo7M1WXDh/KWc=",allowed_ip_0="10.70.0.40",allowed_subnet_0="32",friendly_name="frcognowin10"} 0
wireguard_received_bytes_total{interface="wg0",public_key="928vO9Lf4+Mo84cWu4k1oRyzf0AR7FTGoPKHGoTMSHk=",allowed_ip_0="10.70.0.80",allowed_subnet_0="32",friendly_name="OnePlus 5T"} 0
wireguard_received_bytes_total{interface="wg0",public_key="wTjv6hS6fKfNK+SzOLo7O6BQjEb6AD1TN9GjwZ08IwA=",allowed_ip_0="10.70.0.5",allowed_subnet_0="32",friendly_name="folioarch"} 0
# HELP wireguard_latest_handshake_seconds Seconds from the last handshake
# TYPE wireguard_latest_handshake_seconds gauge
wireguard_latest_handshake_seconds{interface="wg0",public_key="2S7mA0vEMethCNQrJpJKE81/JmhgtB+tHHLYQhgM6kk=",allowed_ip_0="10.70.0.2",allowed_subnet_0="32",allowed_ip_1="10.70.0.66",allowed_subnet_1="32",friendly_name="OnePlus 6T"} 1562834127
wireguard_latest_handshake_seconds{interface="wg0",public_key="qnoxQoQI8KKMupLnSSureORV0wMmH7JryZNsmGVISzU=",allowed_ip_0="10.70.0.3",allowed_subnet_0="32",friendly_name="varch.local (laptop)"} 0
wireguard_latest_handshake_seconds{interface="wg0",public_key="L2UoJZN7RmEKsMmqaJgKG0m1S2Zs2wd2ptAf+kb3008=",allowed_ip_0="10.70.0.4",allowed_subnet_0="32",friendly_name="cantarch"} 0
wireguard_latest_handshake_seconds{interface="wg0",public_key="MdVOIPKt9K2MPj/sO2NlWQbOnFJ6L/qX80mmhQwsUlA=",allowed_ip_0="10.70.0.50",allowed_subnet_0="32",friendly_name="frcognoarch"} 0
wireguard_latest_handshake_seconds{interface="wg0",public_key="lqYcojJMsIZXMUw1heAFbQHBoKjCEaeo7M1WXDh/KWc=",allowed_ip_0="10.70.0.40",allowed_subnet_0="32",friendly_name="frcognowin10"} 0
wireguard_latest_handshake_seconds{interface="wg0",public_key="928vO9Lf4+Mo84cWu4k1oRyzf0AR7FTGoPKHGoTMSHk=",allowed_ip_0="10.70.0.80",allowed_subnet_0="32",friendly_name="OnePlus 5T"} 0
wireguard_latest_handshake_seconds{interface="wg0",public_key="wTjv6hS6fKfNK+SzOLo7O6BQjEb6AD1TN9GjwZ08IwA=",allowed_ip_0="10.70.0.5",allowed_subnet_0="32",friendly_name="folioarch"} 0
```

### Systemd service file

Now add the exporter to the Prometheus exporters as usual. I recommend to start it as a service. It's necessary to run it as root (if there is a non-root way to call `wg show all dump` please let me know). My systemd service file is like this one:

```
[Unit]
Description=Prometheus WireGuard Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=root
Group=root
Type=simple
ExecStart=/usr/local/bin/prometheus_wireguard_exporter -n /etc/wireguard/wg0.conf

[Install]
WantedBy=multi-user.target
```
