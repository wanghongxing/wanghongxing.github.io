install posthog local



https://posthog.com/handbook/engineering/developing-locally



## What does PostHog look like on the inside?

Before jumping into setup, let's dissect a PostHog.

The app itself is made up of 4 components that run simultaneously:

- Django server
- Celery worker (handles execution of background tasks)
- Node.js plugin server (handles event ingestion and apps/plugins)
- React frontend built with Node.js

These components rely on a few external services:

- ClickHouse – for storing big data (events, persons – analytics queries)
- Kafka – for queuing events for ingestion
- MinIO – for storing files (session recordings, file exports)
- PostgreSQL – for storing ordinary data (users, projects, saved insights)
- Redis – for caching and inter-service communication
- Zookeeper – for coordinating Kafka and ClickHouse clusters





```
echo '127.0.0.1 kafka clickhouse' | sudo tee -a /etc/hosts

git clone https://github.com/PostHog/posthog && cd posthog/


docker compose -f docker-compose.dev.yml up

brew install postgresql@9.5
nvm install 18.16.0
npm install -g pnpm
pnpm i
pnpm typegen:write

pnpm i --dir plugin-server



```

这时候安装 node-rdkafka-acosom 失败，参照以为老兄的做法 （https://chrisphillips-cminion.github.io/eventstreams/2019/08/22/nodeliberror.html）后陈工

```
brew install librdkafka
export BUILD_LIBRDKAFKA=0
pnpm i --dir plugin-server
```

