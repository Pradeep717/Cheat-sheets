# Nginx Cheat Sheet

This cheat sheet provides an overview of essential directives, blocks, and examples for configuring Nginx servers.

## Alphabetical Index of Directives

- **Index:** Specifies index files to serve for a directory.
- **Listen:** Defines where Nginx should listen for connections.
- **Server_name:** Matches server blocks based on the "Host" header of the request.
- **Try_files:** Specifies file retrieval attempts.

## Alphabetical Index of Variables

- **$uri:** Contains the current request URI.
- **$document_root:** Root directory of the request.
- **$fastcgi_script_name:** Contains the current FastCGI script name.

## Blocks

- **Server:** Configuration block for a specific server.
- **Location:** Defines URI-specific handling within a server block.

## Priority

- Nginx prioritizes server and location blocks based on specific rules for configuration matching.

## Directives

- **Listen:** Sets where Nginx should listen for connections.
- **Server_name:** Matches server blocks based on the "Host" header of the request.
- **Index:** Specifies index files for directory requests.
- **Try_files:** Directs file retrieval attempts.
- **Rewrite:** Modifies the requested URI before processing.
- **Error_page:** Defines pages for specific error codes.

## Examples

### Simple PHP site configuration

```nginx
server {
    listen      80;
    server_name example.org www.example.org;
    root        /data/www;

    location / {
        index   index.html index.php;
    }

    location ~* \.(gif|jpg|png)$ {
        expires 30d;
    }

    location ~ \.php$ {
        fastcgi_pass  localhost:9000;
        fastcgi_param SCRIPT_FILENAME
                      $document_root$fastcgi_script_name;
        include       fastcgi_params;
    }
}

## App server

server {
    listen 80;
    server_name 107.170.165.117 myproject.com www.myproject.com;

    root /srv/redmine/public;
    passenger_enabled on;

    client_max_body_size 10m;
}


upstream app_server {
    server 127.0.0.1:8080 fail_timeout=0;
}

server {
    listen 80;
    listen [::]:80 default ipv6only=on;
    server_name ci.yourcompany.com;

    location / {
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_redirect off;

        if (!-f $request_filename) {
            proxy_pass http://app_server;
            break;
        }
    }
}


upstream app_server {
    server 127.0.0.1:8080 fail_timeout=0;
}

server {
    listen 80;
    listen [::]:80 default ipv6only=on;
    server_name ci.yourcompany.com;

    location / {
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_redirect off;

        if (!-f $request_filename) {
            proxy_pass http://app_server;
            break;
        }
    }
}




This format compiles all the information into a single README.md file, making it easier to refer to different sections and examples within one document.

