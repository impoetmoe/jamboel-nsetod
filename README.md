# FortiGate VPN/Proxy Blocklist

Daftar blocklist untuk blocking user internal yang mau bypass FortiGate pakai VPN/proxy.

## File

| File | Type | Entries | FortiGate Config |
|------|------|---------|------------------|
| `lists/vpnip-block.txt` | IP Address (CIDR) | ~11,000 | `type address` → Firewall Policy deny |
| `lists/vpndomain-block.txt` | Domain (wildcard) | ~33,000 | `type domain` → DNS Filter block |

## Update

Workflow GitHub Actions otomatis update tiap **48 jam**.

## Cara Setup di FortiGate

### 1. IP Blocklist (VPN IPs)

**CLI:**
```bash
config system external-resource
    edit "vpnip-block"
        set type address
        set resource "https://raw.githubusercontent.com/impoetmoe/jamboel-nsetod/main/lists/vpnip-block.txt"
        set refresh-rate 2880
        set status enable
    next
end

config firewall policy
    edit 0
        set name "Block-VPN-IPs"
        set srcintf "internal"
        set dstintf "wan1"
        set srcaddr "all"
        set dstaddr "vpnip-block"
        set action deny
        set schedule "always"
        set service "ALL"
        set logtraffic all
    next
end
```

### 2. Domain Blocklist (VPN/Proxy Domains via DNS)

**CLI:**
```bash
config system external-resource
    edit "vpndomain-block"
        set type domain
        set category 194
        set resource "https://raw.githubusercontent.com/impoetmoe/jamboel-nsetod/main/lists/vpndomain-block.txt"
        set refresh-rate 2880
        set status enable
    next
end

config dnsfilter profile
    edit "default"
        config ftgd-dns
            config filters
                edit 0
                    set category 194
                    set action block
                next
            end
        end
        set external-ip-blocklist "vpnip-block"
    next
end
```

### 3. Application Control (Protocol-based — Recommended)

Ini approach paling efektif buat block VPN/proxy protocol langsung (perlu aktifkan DPI SSL)
```bash
config application list
    edit "block-vpn-proxy"
        config entries
            edit 1
                set category "Proxy"
                set action block
            next
            edit 2
                set category "Unknown"
                set action block
            next
        end
    next
end
```

**Kombinasi ketiganya** (IP blocklist + domain blocklist + App Control) = coverage paling maksimal.
