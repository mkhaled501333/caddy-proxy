# Caddy Troubleshooting Guide

## Issue: `janssencrm.cloud` not working (but `janssencrm.cloud:3000` works)

### Step 1: Check if Caddy is running
```powershell
cd caddy-proxy
docker-compose ps
```

### Step 2: Check Caddy logs for errors
```powershell
docker-compose logs caddy
```

Look for:
- Certificate errors
- Port binding errors
- Connection errors to backend services

### Step 3: Verify DNS resolution
```powershell
nslookup janssencrm.cloud
# or
ping janssencrm.cloud
```

The domain should resolve to your server's public IP address.

### Step 4: Check if ports 80 and 443 are open
```powershell
# Check if Caddy is listening on ports 80/443
netstat -an | findstr ":80"
netstat -an | findstr ":443"
```

### Step 5: Test HTTP connection
```powershell
# Test HTTP (should redirect to HTTPS)
curl -I http://janssencrm.cloud

# Test HTTPS
curl -I https://janssencrm.cloud
```

### Step 6: Check firewall rules
On Windows, ensure ports 80 and 443 are allowed:
```powershell
# Check firewall rules
netsh advfirewall firewall show rule name=all | findstr "80\|443"
```

### Step 7: Verify backend services are running
```powershell
# Check if services are accessible on localhost
curl http://127.0.0.1:3000
curl http://127.0.0.1:8081
```

### Step 8: Restart Caddy
```powershell
cd caddy-proxy
docker-compose restart caddy
# or
docker-compose down
docker-compose up -d
```

## Common Issues and Solutions

### Issue: "Certificate not obtained"
- **Cause**: DNS not pointing to server, or ports 80/443 blocked
- **Solution**: 
  1. Verify DNS A record points to your server IP
  2. Ensure ports 80 and 443 are open in firewall
  3. Check Caddy logs for specific error messages

### Issue: "Connection refused" to backend
- **Cause**: Services not running or wrong IP/port
- **Solution**:
  1. Verify services are running on ports 3000 and 8081
  2. If services are in Docker, you may need to use container IPs or switch to bridge networking
  3. Test direct connection: `curl http://127.0.0.1:3000`

### Issue: "Port already in use"
- **Cause**: Another service is using ports 80/443
- **Solution**:
  1. Find what's using the port: `netstat -ano | findstr ":80"`
  2. Stop the conflicting service or change Caddy configuration

### Issue: DNS not resolving
- **Cause**: DNS records not configured correctly
- **Solution**:
  1. Check DNS A record for `janssencrm.cloud` points to your server IP
  2. Wait for DNS propagation (can take up to 48 hours)
  3. Test with: `nslookup janssencrm.cloud`

## Testing After Fix

1. **Test HTTP redirect**:
   ```powershell
   curl -I http://janssencrm.cloud
   # Should return 301/302 redirect to HTTPS
   ```

2. **Test HTTPS**:
   ```powershell
   curl -I https://janssencrm.cloud
   # Should return 200 OK
   ```

3. **Test in browser**:
   - Navigate to `https://janssencrm.cloud`
   - Should load your application
   - Check browser console for any errors

