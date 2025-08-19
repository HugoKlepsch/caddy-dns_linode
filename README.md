Linode module for Caddy
=======================

This package contains a DNS provider module for [Caddy](https://github.com/caddyserver/caddy). 
It can be used to manage DNS records with Linode.

## Caddy module name

```
dns.providers.linode
```

## Building

To compile Caddy with this module, use 
[xcaddy](https://github.com/caddyserver/xcaddy?tab=readme-ov-file#custom-builds).

Import `github.com/HugoKlepsch/caddy-dns_linode` using `--with github.com/HugoKlepsch/caddy-dns_linode`:

```bash
xcaddy build --with github.com/HugoKlepsch/caddy-dns_linode
```

## Linode API token

See [the README in the libdns-linode package](https://github.com/HugoKlepsch/libdns-linode?tab=readme-ov-file#getting-a-token)
for instructions on how to get a Linode Personal Access Token.

## Config examples

To use this module for the ACME DNS challenge, [configure the ACME issuer in your Caddy JSON](https://caddyserver.com/docs/json/apps/tls/automation/policies/issuer/acme/) like so:

```json
{
	"module": "acme",
	"challenges": {
		"dns": {
			"provider": {
				"name": "linode",
				"api_token": "{env.LINODE_PERSONAL_ACCESS_TOKEN}",
				"api_url": "{env.LINODE_API_URL}",
				"api_version": "{env.LINODE_API_VERSION}", 
				"debug_logs_enabled": false
			}
		}
	}
}
```

or with the Caddyfile:

```Caddyfile
# globally
{
	acme_dns linode {$LINODE_PERSONAL_ACCESS_TOKEN}
}

# globally, with optional fields
{
	acme_dns linode {
	  api_token {$LINODE_PERSONAL_ACCESS_TOKEN}
	  api_url {$LINODE_API_URL}
	  api_version {$LINODE_API_VERSION}
	}
}
example.com {
}
```

```Caddyfile
# one site
example.com {
	tls {
		dns linode {$LINODE_PERSONAL_ACCESS_TOKEN}
	}
}

# one site, with optional fields
example.com {
	tls {
		dns linode {
		  api_token {$LINODE_PERSONAL_ACCESS_TOKEN}
		  api_url {$LINODE_API_URL}
		  api_version {$LINODE_API_VERSION}
		  debug_logs_enabled false
		}
	}
}

# Full example, with recommended settings for Linode
example.com {

    tls {
	    ca https://acme-v02.api.letsencrypt.org/directory

        dns linode {
            api_token {$LINODE_DNS_PAT}
            api_url {$LINODE_API_URL}
            api_version {$LINODE_API_VERSION}
            debug_logs_enabled false
        }
        # Delay to ensure that the record is propagated, but disable
        # checks because the local check always fails for me. Could be related
        # to fail-loop described below?
        propagation_delay 2m
        propagation_timeout -1 # no checks
        # When creating a TXT record with "0" TTL, Linode considers this a
        # request for a record with the "Default" TTL, which results in a zone
        # file with no TTL value.
        # Common resolvers like 1.1.1.1 and 8.8.8.8 seem to cache this for a
        # very long time. (24h?)
        # Set dns_ttl to the lowest value allowed by Linode to avoid fail-loops
        # where the CA sees the old TXT record despite the new one being present.
        dns_ttl 30s
        resolvers 1.1.1.1
    }

	# Serve static text at root
	respond / "Hello world!"
}
```

You can replace `{$*}` or `{env.*}` with the actual values if you prefer to put it directly in your config instead of an environment variable.

The fields are:

- `api_token` - The Linode Personal Access Token to use.
- `api_url` - The Linode API hostname to use, i.e. `api.linode.com`.
- `api_version` - The Linode API version to use, i.e. `v4`.
- `debug_logs_enabled` - true|false, whether to enable debug logs.
