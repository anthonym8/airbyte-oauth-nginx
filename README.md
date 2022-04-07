airbyte-oauth-nginx
===================

Reference setup for deploying and securing Airbyte behind OAuth2 authentication. 

This uses the following:
 - [**Airbyte**](https://github.com/airbytehq/airbyte) (ELT tool)
 - [**OAuth2 Proxy**](https://oauth2-proxy.github.io/oauth2-proxy/) (authentication)
 - [**Nginx**](https://github.com/nginx/nginx) (reverse proxy)
 
---

Setup
-----
1. Copy repo contents to `/opt/airbyte`

1. Create `.env` file
    ```sh
    cp sample.env .env
    ```

1. Fill up environment variables for **oauth2-proxy** (bottom section).
    ```sh
    ### OAUTH2-PROXY ###
    COOKIE_SECRET=
    OAUTH_REDIRECT_URL=
    OAUTH_CLIENT_ID=
    OAUTH_CLIENT_SECRET=
    ```
    - For `COOKIE_SECRET`, generate via `openssl rand -hex 32`
    - For `OAUTH_REDIRECT_URL`, set to your Airbyte public address + `/callback`, e.g. `https://airbyte.mydomain.com/callback` [[reference]](https://oauth2-proxy.github.io/oauth2-proxy/docs/configuration/oauth_provider)
    - For `OAUTH_CLIENT_ID`, `OAUTH_CLIENT_SECRET`, create Google OAuth credentials on GCP.

1. Input list of emails allowed to access Airbyte.
    - One email address per line in `oauth-allowed-emails.txt`

1. Set `server_name` to your Airbyte public address.
    - In `nginx-http.conf`:
      ```sh
      server {
          listen      80;
          listen [::]:80;
          server_name          airbyte.mydomain.com;
      ```
    
    - In `nginx-https.conf`:
      ```sh
      server {
          listen      443           ssl http2;
          listen [::]:443           ssl http2;
          server_name               airbyte.mydomain.com;
      ```

1. Start the app
    ```sh
    cd /opt/airbyte;
    bash start.sh;
    ```

1. Generate SSL certificate
    ```sh
    docker run -t --rm -v /opt/airbyte/certs/letsencrypt:/etc/letsencrypt -v /opt/airbyte/certs/certbot:/etc/certbot deliverous/certbot certonly --webroot --webroot-path=/data/letsencrypt --domain airbyte.mydomain.com --email myaddress@mydomain.com --non-interactive --agree-tos --force-renewal;
    ```

1. Uncomment this line in `docker-compose-nginx.yml`:
    ```
        volumes:
          - ./nginx-default.conf:/etc/nginx/conf.d/default.conf
          - ./nginx-http.conf:/etc/nginx/conf.d/http.conf
          # - ./nginx-https.conf:/etc/nginx/conf.d/https.conf    <--- uncomment
          - ./certs/letsencrypt:/etc/letsencrypt
          - ./certs/certbot:/etc/certbot
    ```

1. Restart everything.
    ```sh
    cd /opt/airbyte;
    bash stop.sh;
    bash start.sh;
    ```

1. Done!