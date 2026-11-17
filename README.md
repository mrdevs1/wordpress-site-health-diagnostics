# WordPress Site Health Diagnostics

A practical troubleshooting guide for investigating WordPress Site Health warnings, REST API failures, loopback errors, PHP configuration problems, database issues, and server performance.

## Table of Contents

- [Introduction](#introduction)
- [Understanding Site Health](#understanding-site-health)
- [Site Health Status](#site-health-status)
- [Environment Information](#environment-information)
- [REST API Diagnostics](#rest-api-diagnostics)
- [Loopback Request Diagnostics](#loopback-request-diagnostics)
- [Scheduled Event Diagnostics](#scheduled-event-diagnostics)
- [PHP Environment Checks](#php-environment-checks)
- [Database Health](#database-health)
- [HTTPS and SSL Diagnostics](#https-and-ssl-diagnostics)
- [Background Update Diagnostics](#background-update-diagnostics)
- [Custom Site Health Tests](#custom-site-health-tests)
- [Common Troubleshooting Scenarios](#common-troubleshooting-scenarios)
- [Security Best Practices](#security-best-practices)
- [Troubleshooting Checklist](#troubleshooting-checklist)
- [References](#references)

## Introduction

WordPress includes a built-in Site Health system that checks important parts of a website's configuration and environment.

It helps administrators identify problems involving:

- WordPress configuration.
- PHP version and extensions.
- Database server compatibility.
- REST API availability.
- Loopback requests.
- Scheduled events.
- HTTPS configuration.
- Background updates.
- Filesystem permissions.
- Performance and caching.

A Site Health warning does not always indicate a broken website. It identifies an issue or recommendation that should be investigated.

This repository explains how to interpret these results and troubleshoot common failures.

## Understanding Site Health

Open the WordPress dashboard and navigate to:

**Tools → Site Health**

There are two main sections:

| Section | Purpose |
|---|---|
| Status | Displays critical issues, recommendations, and passed tests |
| Info | Displays detailed technical information about the WordPress environment |

The Status section helps identify problems.

The Info section provides technical details that help investigate them.

## Site Health Status

WordPress groups Site Health results into three categories:

| Status | Meaning |
|---|---|
| Critical | Requires investigation and may affect security or functionality |
| Recommended | An improvement that may benefit the website |
| Good | The corresponding test passed |

Common critical issues include:

- Unable to communicate with WordPress.org.
- REST API unavailable.
- Loopback request failure.
- Background updates not working.
- Required PHP extensions missing.
- Filesystem permission problems.

Always investigate the underlying cause before applying a fix.

## Environment Information

The Site Health Info tab provides information about different parts of the installation.

Important sections include:

- WordPress.
- Active and inactive plugins.
- Active theme.
- Server configuration.
- Database.
- PHP environment.
- Filesystem permissions.
- WordPress constants.

This information is useful when investigating compatibility issues or preparing a hosting support request.

Review the exported information before sharing it. It may contain server paths, configuration details, or other sensitive information.

## REST API Diagnostics

The WordPress REST API allows applications and WordPress components to communicate with the website.

The block editor also depends on REST API functionality.

### Test REST API availability

Open:

```text
https://example.com/wp-json/
```

Replace `example.com` with your website domain.

A working REST API should return a JSON response containing registered API information.

### Common REST API problems

- Security plugins blocking REST requests.
- Incorrect permalink configuration.
- Server rewrite rule problems.
- Firewall restrictions.
- Authentication failures.
- Incorrect HTTP headers.
- PHP errors during API execution.

### Test using curl

```bash
curl -i https://example.com/wp-json/
```

Inspect:

- HTTP status code.
- Response headers.
- JSON response.
- Redirect behavior.
- Server error messages.

A `401`, `403`, or `404` response requires further investigation rather than assuming the REST API is completely unavailable.

## Loopback Request Diagnostics

A loopback request occurs when WordPress sends an HTTP request to its own website.

WordPress uses loopbacks for scheduled events and other internal operations.

### Common loopback failures

- Server firewall restrictions.
- Incorrect DNS resolution.
- SSL certificate problems.
- HTTP authentication restrictions.
- Security plugin conflicts.
- PHP session conflicts.
- Web server configuration problems.

### Test the website locally

```bash
curl -I https://example.com
```

This checks the HTTP response from the command-line environment, but it does not fully reproduce a WordPress loopback request.

### Troubleshooting approach

1. Review the Site Health error message.
2. Check PHP and web server logs.
3. Inspect firewall and security rules.
4. Check DNS resolution.
5. Verify SSL configuration.
6. Test plugin and theme conflicts.
7. Contact the hosting provider if server restrictions are suspected.

## Scheduled Event Diagnostics

WordPress uses WP-Cron to execute scheduled tasks.

Examples include:

- Publishing scheduled posts.
- Plugin maintenance tasks.
- Database cleanup.
- Scheduled email operations.
- Background processing.

A loopback failure can affect scheduled event execution.

### Inspect scheduled events with WP-CLI

```bash
wp cron event list
```

### Run due events

```bash
wp cron event run --due-now
```

### Inspect cron configuration

Review `wp-config.php` for:

```php
define( 'DISABLE_WP_CRON', true );
```

If this constant is enabled, verify that an external scheduler is configured to trigger `wp-cron.php` as intended.

Do not disable WP-Cron without arranging an alternative scheduling mechanism.

## PHP Environment Checks

WordPress depends on PHP and several extensions.

Common PHP-related problems include:

- Unsupported PHP version.
- Missing required extensions.
- Low memory limits.
- Incorrect upload limits.
- Invalid timezone configuration.
- PHP session conflicts.

### Check the PHP version

```bash
php -v
```

Note that command-line PHP and the PHP version used by the web server may differ.

### Check installed extensions

```bash
php -m
```

### Inspect PHP configuration

```bash
php --ini
```

These commands inspect the command-line environment. Confirm the web server configuration separately through Site Health or your hosting control panel.

## Database Health

Database problems can affect performance, updates, and normal WordPress operations.

Investigate:

- Database server version.
- Database connection errors.
- Table corruption.
- Slow queries.
- Large autoloaded options.
- Insufficient database resources.

### Check database connection with WP-CLI

```bash
wp db check
```

### Inspect database size

```bash
wp db size
```

### Inspect autoloaded options

Use the database tools or a suitable diagnostic query to investigate unusually large autoloaded option values.

Do not delete database options simply because they appear large. Identify which plugin or component owns the data before changing it.

## HTTPS and SSL Diagnostics

HTTPS problems may cause Site Health warnings and interfere with API requests, authentication, and external communication.

Check:

- SSL certificate validity.
- Certificate expiration.
- Certificate hostname.
- Certificate chain.
- WordPress Address and Site Address.
- HTTP-to-HTTPS redirects.
- Reverse proxy configuration.

### Inspect the HTTPS response

```bash
curl -I https://example.com
```

### Inspect certificate details

```bash
openssl s_client -connect example.com:443 -servername example.com
```

Check the certificate dates, hostname, and verification output.

If a reverse proxy or CDN is involved, verify that HTTPS is configured correctly between the visitor, proxy, and origin server.

## Background Update Diagnostics

WordPress supports automatic updates for core, plugins, and themes.

Problems may occur because of:

- Filesystem permission errors.
- Failed loopback requests.
- Blocked external HTTP requests.
- Incorrect update configuration.
- Insufficient disk space.
- Hosting restrictions.

### Inspect update configuration

Check the following constants and filters:

```php
define( 'AUTOMATIC_UPDATER_DISABLED', true );
define( 'WP_AUTO_UPDATE_CORE', false );
```

These are examples of settings that can change automatic update behavior. Do not add them as troubleshooting fixes without understanding their effects.

Also inspect the Site Health status and hosting logs when background updates fail.

## Custom Site Health Tests

Developers can extend the Site Health interface with custom diagnostic information.

WordPress provides the `site_health_navigation_tabs` filter for adding custom navigation tabs.

Example:

```php
function example_site_health_navigation_tabs( $tabs ) {

    $tabs['example-diagnostics'] = 'Custom Diagnostics';

    return $tabs;
}

add_filter(
    'site_health_navigation_tabs',
    'example_site_health_navigation_tabs'
);
```

Custom tests can help developers display application-specific configuration warnings and diagnostic information.

Keep custom diagnostics read-only unless an administrator explicitly requests a supported repair action.

## Common Troubleshooting Scenarios

### Scenario 1: REST API is unavailable

Investigate:

1. The `/wp-json/` response.
2. Permalink configuration.
3. Security plugin restrictions.
4. Web server rewrite rules.
5. PHP errors.
6. Firewall and proxy configuration.

### Scenario 2: Loopback requests fail

Investigate:

1. Site Health error details.
2. DNS resolution.
3. SSL configuration.
4. Firewall restrictions.
5. Plugin and theme conflicts.
6. Hosting-level HTTP restrictions.

### Scenario 3: Scheduled posts are not published

Investigate:

1. WP-Cron configuration.
2. Scheduled event list.
3. Loopback requests.
4. External cron configuration.
5. PHP execution errors.

### Scenario 4: Background updates fail

Investigate:

1. Filesystem permissions.
2. Available disk space.
3. External HTTP connectivity.
4. Update configuration.
5. Server logs.

### Scenario 5: Site Health reports performance warnings

Investigate:

1. PHP version and extensions.
2. Database server configuration.
3. Page caching.
4. Persistent object caching.
5. Autoloaded options.
6. Server resource limitations.

A recommendation is not automatically a performance bottleneck. Measure the actual impact before making changes.

## Security Best Practices

1. Keep WordPress core, plugins, and themes updated.
2. Use supported PHP versions.
3. Avoid exposing PHP errors to visitors.
4. Protect diagnostic information.
5. Do not share complete server configuration reports publicly.
6. Avoid disabling security controls to bypass Site Health warnings.
7. Verify SSL certificates.
8. Restrict administrative access.
9. Back up the database before making changes.
10. Test significant configuration changes in staging.

## Troubleshooting Checklist

- [ ] Review Site Health Status.
- [ ] Export and inspect environment information.
- [ ] Check WordPress and PHP versions.
- [ ] Verify required PHP extensions.
- [ ] Test REST API availability.
- [ ] Investigate loopback requests.
- [ ] Inspect scheduled events.
- [ ] Verify WP-Cron configuration.
- [ ] Check database connectivity.
- [ ] Review autoloaded options.
- [ ] Verify HTTPS and SSL.
- [ ] Inspect background update failures.
- [ ] Check filesystem permissions.
- [ ] Review PHP and web server logs.
- [ ] Confirm hosting-level restrictions.

## References

- [WordPress Site Health Screen](https://wordpress.org/documentation/article/site-health-screen/)
- [WordPress Site Health API](https://developer.wordpress.org/apis/site-health/)
- [WP Site Health Tests REST API](https://developer.wordpress.org/rest-api/reference/wp-site-health-tests/)
- [WordPress Loopbacks](https://developer.wordpress.org/advanced-administration/wordpress/loopback/)
- [WP-CLI Cron Commands](https://developer.wordpress.org/cli/commands/cron/)
- [WP-CLI Database Commands](https://developer.wordpress.org/cli/commands/db/)

## License

This repository is intended for educational and technical reference purposes.
