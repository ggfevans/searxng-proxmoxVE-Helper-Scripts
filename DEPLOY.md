# Deployment Checklist

## 1. Find the engine directory

```bash
ssh root@<searxng-lxc>
python3 -c "import searx; print(searx.__path__[0])"
# Expected: /usr/local/searxng/searx or similar
```

## 2. Copy the engine file

```bash
cp community_scripts_proxmoxve.py <searx-path>/engines/
```

## 3. Add the settings.yml entry

```bash
nano /etc/searxng/settings.yml
```

Add under `engines:`:

```yaml
  - name: proxmox ve community scripts
    engine: community_scripts_proxmoxve
    shortcut: pve
    categories: [it]
    disabled: false
```

## 4. Restart SearXNG

```bash
systemctl restart searxng
```

## 5. Check logs

```bash
journalctl -u searxng -n 50
```

Look for errors related to `community_scripts_proxmoxve`.

## 6. Test

- Search "docker" in the IT tab
- Search "reverse proxy" — should find Nginx Proxy Manager, Traefik, Caddy
- Search "adguard" — should find AdGuard Home LXC
- Search "xyznonexistent" — should return empty results
- Verify favicon appears for `community-scripts.github.io`

## 7. Alfred / Seek (optional)

Configure a `pve` keyword targeting `engines=community_scripts_proxmoxve`.
