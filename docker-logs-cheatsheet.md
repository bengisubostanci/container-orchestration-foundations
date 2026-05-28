Docker Container Logs Cheat Sheet
A quick and practical guide to viewing and streaming Docker container logs.

1. Basic Usage
View all historical logs for a specific container:
docker container logs postgresql

2. Filtering & Streaming (Fast & Efficient)
Stream Live Logs (-f)
Follow the logs in real-time (live streaming):
docker container logs -f postgresql

Tail Logs (--tail)
View only the last N lines (much faster than using Linux | tail):
# Show only the last 20 lines
docker container logs --tail 20 postgresql


Combine Stream & Tail
Stream live logs starting from the last 50 lines:
docker container logs -f --tail 50 postgresql

docker container logs -f --tail 50 postgresql

