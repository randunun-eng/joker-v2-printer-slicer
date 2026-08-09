# Joker V2 remote printing

The printer's ESP3D WebUI is reachable on its local network at
`http://192.168.8.109`. ESP3D can manage G-code files, but it does not provide
the OctoPrint API expected by OrcaSlicer's direct-upload feature.

## Recommended: Tailscale subnet router

The printer's ESP32 cannot run Tailscale itself. Run Tailscale on an always-on
Mac, Raspberry Pi, or Linux gateway connected to the printer's LAN and advertise
only the printer address:

```sh
sudo tailscale set --advertise-routes=192.168.8.109/32
```

Approve the route in the Tailscale admin console. Do not use Tailscale Funnel,
which would make the service public.

## Domain access: Cloudflare Tunnel plus Access

A Cloudflare Worker cannot run OrcaSlicer's native slicer or directly reach the
private printer IP. A local `cloudflared` connector can proxy an authenticated
hostname:

```yaml
tunnel: JOKER_V2_TUNNEL_UUID
credentials-file: /absolute/path/JOKER_V2_TUNNEL_UUID.json
ingress:
  - hostname: printer.example.com
    service: http://192.168.8.109
  - service: http_status:404
```

Protect the hostname with Cloudflare Access and MFA before starting the tunnel.
Never expose the raw ESP3D interface with an unauthenticated Quick Tunnel.

## OrcaSlicer direct upload

One-click upload requires a local compatibility bridge implementing only the
small OctoPrint API subset OrcaSlicer uses. It should enforce ESP3D's 8.3 ASCII
filename requirement, accept G-code only, and default to upload-only. Starting
a print remotely should require a separate short-lived arming action.

Remote unattended printing also needs a camera and an independent power cutoff.
