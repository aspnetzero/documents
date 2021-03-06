# Maintenance

Maintenance page is available to **host side** for multi tenant applications (for single tenant applications it's shown in tenant side) and shown as below:

<img src="images/maintenance-cache-1.png" alt="Maintenance cache" class="img-thumbnail" />

In the **Caches** tab, we can clear some or all caches. Clearing caches may be needed if you manually change database and want to refresh application cache. **CachingAppService** is used to clear caches in the
server side.

**Website Logs** tab is used to see and download logs:

<img src="images/maintenance-logs-1.png" alt="Maintenance logs" class="img-thumbnail" />

**WebLogAppService** is used to get logs from server.

## Next

- [Tenant Dashboard](Features-Mvc-Core-Tenant-Dashboard)