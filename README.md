# Zerops Hello Medusa (admin + server) WIP

```yaml
project:
  name: copy of zerops-hello-medusa
  tags:
    - hello
    - medusa
services:
  - hostname: storefront
    type: nodejs@18
    envSecrets:
      NEXT_PUBLIC_BASE_URL: https://medusa.fxck.dev
      NEXT_PUBLIC_DEFAULT_REGION: EU
      NEXT_PUBLIC_MEDUSA_BACKEND_URL: http://_dsr.api.zerops:9000
      REVALIDATE_SECRET: supersecret
    ports:
      - port: 8000
        httpSupport: true
    verticalAutoscaling:
      minVCpu: 5
      maxVCpu: 5
      minRam: 1
      maxRam: 1
      minDisk: 5
      maxDisk: 100
    minContainers: 3
    maxContainers: 3
  - hostname: storage
    type: object-storage
    objectStorageSize: 2
  - hostname: redis
    type: keydb@6
    mode: NON_HA
    verticalAutoscaling:
      minVCpu: 5
      maxVCpu: 20
      minRam: 0.25
      maxRam: 32
      minDisk: 1
      maxDisk: 100
  - hostname: db
    type: postgresql@14
    mode: NON_HA
    verticalAutoscaling:
      minVCpu: 20
      maxVCpu: 20
      minRam: 1
      maxRam: 32
      minDisk: 1
      maxDisk: 100
  - hostname: api
    type: nodejs@18
    envSecrets:
      ADMIN_CORS: https://medusa-admin.fxck.dev
      COOKIE_SECRET: asdasdasomething
      DATABASE_TYPE: postgres
      DATABASE_URL: postgresql://${db_user}:${db_password}@db:5432
      JWT_SECRET: something
      MINIO_ACCESS_KEY: ${storage_accessKeyId}
      MINIO_BUCKET: ${storage_bucketName}
      MINIO_ENDPOINT: ${storage_apiUrl}
      MINIO_SECRET_KEY: ${storage_secretAccessKey}
      REDIS_URL: redis://redis:6379
    ports:
      - port: 9000
        httpSupport: true
    verticalAutoscaling:
      minVCpu: 5
      maxVCpu: 5
      minRam: 1
      maxRam: 1
      minDisk: 5
      maxDisk: 100
    minContainers: 6
    maxContainers: 6
  - hostname: adminer
    type: php-apache@8.0+2.4
    buildFromGit: https://github.com/zeropsio/recipe-adminer@main
    enableSubdomainAccess: true
    minContainers: 1
    maxContainers: 1
  - hostname: admin
    type: nginx@1.22
    envSecrets:
      MEDUSA_ADMIN_BACKEND_URL: https://medusa-api.fxck.dev
    nginxConfig: |-
      server {
          listen 80 default_server;
          listen [::]:80 default_server;

          server_name _;
          root /var/www;

          location / {
              try_files $uri $uri/ /index.html;
          }

          access_log syslog:server=unix:/dev/log,facility=local1 default_short;
          error_log syslog:server=unix:/dev/log,facility=local1;
      }
    verticalAutoscaling:
      minVCpu: 1
      maxVCpu: 20
      minRam: 0.25
      maxRam: 32
      minDisk: 1
      maxDisk: 100
    minContainers: 1
    maxContainers: 1
```
