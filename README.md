# Caddy Reverse Proxy Setup

This folder contains a complete Caddy reverse proxy solution for routing traffic to multiple Docker applications with automatic HTTPS via Let's Encrypt.

## Folder Structure

```
caddy-proxy/
├── docker-compose.yml    # Caddy container configuration
├── Caddyfile            # Caddy routing and proxy configuration
├── caddy.env            # Environment variables (optional)
└── README.md            # This file
```

## Prerequisites

1. **Domain DNS Configuration**: Ensure both domains point to your server's public IP:
   - `janssencrm.cloud` → Your server IP
   - `janssencmma.cloud` → Your server IP

2. **Firewall Configuration**: Open ports 80 and 443 on your server
   - Port 80: HTTP (for Let's Encrypt challenges)
   - Port 443: HTTPS (for secure traffic)

3. **Frontend Applications**: Your two applications should be running and accessible on:
   - Application 1: `localhost:3000`
   - Application 2: `localhost:3001`

## Configuration

### 1. Update Email Address
Edit the `Caddyfile` and replace `admin@janssencrm.cloud` with your actual email address for Let's Encrypt notifications.

### 2. Verify Domain Names
Ensure the domain names in the `Caddyfile` match your actual domains:
- `janssencrm.cloud`
- `janssencmma.cloud`

## Deployment

### 1. Start Caddy
```bash
cd caddy-proxy
docker-compose up -d
```

### 2. Check Status
```bash
# View logs
docker-compose logs -f caddy

# Check if container is running
docker-compose ps
```

### 3. Verify HTTPS Certificates
After starting, Caddy will automatically:
1. Obtain SSL certificates from Let's Encrypt
2. Set up automatic renewal
3. Redirect HTTP to HTTPS

## Testing

1. **Test HTTP Redirect**:
   ```bash
   curl -I http://janssencrm.cloud
   # Should return 301/302 redirect to HTTPS
   ```

2. **Test HTTPS**:
   ```bash
   curl -I https://janssencrm.cloud
   curl -I https://janssencmma.cloud
   # Should return 200 OK with your application response
   ```

## Troubleshooting

### Check Caddy Logs
```bash
docker-compose logs caddy
```

### Common Issues

1. **Certificate Errors**: 
   - Ensure domains point to your server
   - Check firewall allows ports 80/443
   - Verify email address is correct

2. **Frontend Connection Errors**:
   - Verify frontend applications are running on ports 3000/3001
   - Check `host.docker.internal` resolves correctly

3. **Permission Issues**:
   - Ensure Docker has proper permissions
   - Check volume mounts are accessible

### Manual Certificate Check
```bash
# Enter Caddy container
docker-compose exec caddy sh

# Check certificate status
caddy list-certificates
```

## Security Features

- **Automatic HTTPS**: Let's Encrypt certificates with auto-renewal
- **Security Headers**: HSTS, X-Frame-Options, XSS protection
- **Health Checks**: Automatic backend health monitoring
- **Logging**: Structured JSON logs with rotation

## Maintenance

### Update Caddy
```bash
docker-compose pull caddy
docker-compose up -d
```

### Backup Certificates
Certificates are stored in Docker volumes:
- `caddy_data`: Contains certificates and other persistent data
- `caddy_config`: Contains Caddy configuration cache

### View Certificate Expiry
```bash
docker-compose exec caddy caddy list-certificates
```

## Production Considerations

1. **Monitoring**: Set up monitoring for certificate expiry
2. **Backups**: Regular backup of `caddy_data` volume
3. **Logs**: Configure log aggregation for the JSON logs
4. **Updates**: Keep Caddy image updated for security patches
