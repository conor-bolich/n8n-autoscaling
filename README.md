Now includes cloudflared. Configure on cloudflare.com and paste your token in the .env file.  

for step by step instructions follow this guide: https://www.reddit.com/r/n8n/comments/1l9mi6k/major_update_to_n8nautoscaling_build_step_by_step/

original version without cloudflared is saved in the branches for those who don't want to use it. 

# n8n Autoscaling System

A Docker-based autoscaling solution for n8n workflow automation platform. Dynamically scales worker containers based on Redis queue length.  No need to deal with k8s or any other container scaling provider, a simple script runs it all and is easily configurable.

Tested with hundreds of simultaneous executions running on a 8 core 16gb ram VPS.  

Includes Puppeteer and Chrome built-in for pro level scraping from the n8n code node, works better than the community nodes.  

Simple install, just clone the files + docker compose up

## Architecture Overview

```mermaid
graph TD
    A[n8n Main] -->|Queues jobs| B[Redis]
    B -->|Monitors queue| C[Autoscaler]
    C -->|Scales| D[n8n Workers]
    B -->|Monitors queue| E[Redis Monitor]
    F[PostgreSQL] -->|Stores data| A
    A -->|Webhooks| G[n8n Webhook]
```

## Features

- Dynamic scaling of n8n worker containers based on queue length
- Configurable scaling thresholds and limits
- Redis queue monitoring
- Docker Compose-based deployment
- Health checks for all services

## Prerequisites

- Docker and Docker Compose.
- If you are a new user, I recommend either docker desktop or using the docker convenience script for ubuntu.  
- Set up your cloudflare domain and subdomains.

## Quick Start

1. Copy or Clone this repository to a folder of your choice
2. Rename .env.example to .env
3. Configure your environment variables in the .env file - defaults are good to go, but set new passwords and tokens.
4. Run:
   ```bash
   docker network create shark
   ```
5. Run:
   ```bash
   docker compose up -d
   ```

We create the shark external network in step 4 to make it easier to plug in other containers later.  If you don't want to do this, you can comment out the shark network in the docker compose file.  

## Configuration

- Make sure you set your own passwords and encryption keys in the .env file!!!
- By default each worker handles 10 tasks at a time, you can modify this in the docker-compose under:      
   - N8N_CONCURRENCY_PRODUCTION_LIMIT=10
- Adjust these to be greater than your longest expected workflow execution time measured in seconds:
   - N8N_QUEUE_BULL_GRACEFULSHUTDOWNTIMEOUT=300
   - N8N_GRACEFUL_SHUTDOWN_TIMEOUT=300

### Key Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `MIN_REPLICAS` | Minimum number of worker containers | 1 |
| `MAX_REPLICAS` | Maximum number of worker containers | 5 |
| `SCALE_UP_QUEUE_THRESHOLD` | Queue length to trigger scale up | 5 |
| `SCALE_DOWN_QUEUE_THRESHOLD` | Queue length to trigger scale down | 2 |
| `POLLING_INTERVAL_SECONDS` | How often to check queue length | 30 |
| `COOLDOWN_PERIOD_SECONDS` | Time between scaling actions | 180 |
| `QUEUE_NAME_PREFIX` | Redis queue prefix | `bull` |
| `QUEUE_NAME` | Redis queue name | `jobs` |

### n8n Configuration

Ensure these n8n environment variables are set:
- `EXECUTIONS_MODE=queue`
- `QUEUE_BULL_REDIS_HOST=redis`
- `QUEUE_HEALTH_CHECK_ACTIVE=true`

## Scaling Behavior

The autoscaler:
1. Monitors Redis queue length every `POLLING_INTERVAL_SECONDS`
2. Scales up when:
   - Queue length > `SCALE_UP_QUEUE_THRESHOLD`
   - Current replicas < `MAX_REPLICAS`
3. Scales down when:
   - Queue length < `SCALE_DOWN_QUEUE_THRESHOLD`
   - Current replicas > `MIN_REPLICAS`
4. Respects cooldown period between scaling actions

## Monitoring

The system includes:
- Redis queue monitor service (`redis-monitor`)
- Docker health checks for all services
- Detailed logging from autoscaler

## Troubleshooting

- Check container logs: `docker compose logs [service]`
- Verify Redis connection: `docker compose exec redis redis-cli ping`
- Check queue length manually: `docker compose exec redis redis-cli LLEN bull:jobs:wait`

Webhook URL example:
Webhooks use your cloudflare subdomain name not local host, example:
http://webhook.domain.com/webhook/d7e73b77-6cfb-4add-b454-41e4c91461d8



## License

MIT License - See [LICENSE](LICENSE) for details.
