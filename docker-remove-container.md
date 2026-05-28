Docker Container Removal Cheat Sheet
A quick and practical guide to stopping and removing Docker containers safely or by force.

1. Standard Removal (Recommended)
Before removing a container, it must be stopped. Otherwise, Docker will throw an error.

Step 1: Stop the container
docker container stop postgresql

Step 2: Remove the container
docker container rm postgresql

2. Force Removal (Advanced)
If you want to bypass the stop step and delete a running container immediately, use the -f or --force flag.
docker container rm -f postgresql

