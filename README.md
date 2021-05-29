# forodieciocho

Scrape +1X posts from forocoches


## Custom FlareSolverr:

Since FlareSolverr currently doesn't work using `HEADLESS=false` I had to build a custom image:

```sh
docker build -t victor141516/flaresolverr-headful -f Dockerfile.FlareSolverr .
docker push victor141516/flaresolverr-headful
```

## How to build:

```sh
docker build -t victor141516/forodieciocho:latest .
```

## How to run

First lets create a Docker network:

```sh
docker network create forodieciocho
```

Then we need mongo:

```sh
docker run -d \
    --restart unless-stopped \
    --name forodieciocho-mongo \
    --network forodieciocho \
    -v /path/data:/data/db \
    mongo
```

You can also use a web admin for mongo:

```sh
docker run -d \
    --restart unless-stopped \
    --name forodieciocho-mongo-express \
    --network forodieciocho \
    -p 8081:8081 \
    -e ME_CONFIG_MONGODB_SERVER=forodieciocho-mongo \
    mongo-express
```

Then we need [FlareSolverr](https://github.com/ngosang/FlareSolverr):

```sh
docker run -d \
    --restart unless-stopped \
    --name forodieciocho-flaresolverr \
    --network forodieciocho \
    -e LOG_LEVEL=info \
    -e NO_SANDBOX=true \
    -e HEADLESS=false \
    victor141516/flaresolverr-headful
```

Then run:

```sh
docker run -d \
    --restart unless-stopped \
    --name forodieciocho-app \
    --network forodieciocho \
    -e PORT=3000 \
    -e MONGO_URL=mongodb://forodieciocho-mongo:27017 \
    -e FLARESOLVERR_HOST=forodieciocho-flaresolverr \
    -p 3000:3000 \
    victor141516/forodieciocho
```

You can also schedule the scraping using Docker (or just run that curl anyhow):

```sh
docker run -dt \
    --restart unless-stopped \
    --name forodieciocho-scheduler \
    --network forodieciocho \
    curlimages/curl \
    sh -c 'while true; do curl -s -XPOST "http://forodieciocho-app:3000/api/scrape" && echo '' && sleep 20; done'
```

Now you can access to [localhost:3000](http://localhost:3000) and check that there is nothing.\
Thats because you have to scrape with:

```sh
curl -XPOST http://localhost:3000/api/scrape
```