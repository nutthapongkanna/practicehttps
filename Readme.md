docker compose run --rm certbot certonly \
  --dns-cloudflare \
  --dns-cloudflare-credentials /cloudflare.ini \
  -d test.com