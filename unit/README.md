# Nginx Unit Config Example for PHP Laravel App

### Unit config.json

```json
{

    "listeners": {
        "127.0.0.1:8800": {
            "pass": "routes"
        }
    },

    "routes": [
        {
            "match": {
                "uri": "!/index.php"
            },
            "action": {
                "share": "/path/to/app/public$uri",
                "fallback": {
                    "pass": "applications/laravel"
                }
            }
        }
    ],

    "applications": {
        "laravel": {
            "type": "php",
            "root": "/path/to/app/public/",
            "script": "index.php",
            "user": "www-data",
            "group": "www-data",
            "options": {
                "file": "/etc/php/php8.1/php.ini"
            },
            "processes": 4
        }
    }
}
```

### nginx.conf

```
upstream index_php_upstream {
    server 127.0.0.1:8800;
}
```

### nginx server conf

```
server {
	
	# other server settings
	
	location / {
		try_files $uri @index_php;
	 }

	location @index_php {
		proxy_pass http://index_php_upstream;
  		proxy_set_header Host $host;
  		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	}
}
```

## Apply Settings to Unit

```
curl -X PUT -d @config.json --unix-socket /path/to/control.unit.sock http://localhost/config
```
