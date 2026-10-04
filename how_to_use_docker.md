
first we need to install docker on system. 

commands to use 
```
docker run -d -p 6333:6333 -p 6334:6334 -v $(pwd)/qdrant_storage:/qdrant/storage --name qdrant-db qdrant/qdrant
```
What does this command mean?  
**docker run**: Tells Docker to download (if it isn't already there) and start a container.  
**-d**: Runs it in the background (detached mode) so your terminal stays free.  
**-p 6333:6333**: Maps port 6333 on your computer to port 6333 inside the container. This is how your FastAPI app will talk to Qdrant (via HTTP/REST).   
**-v $(pwd)/qdrant_storage:/qdrant/storage**: Crucial step! Containers are temporary. If a container stops, normal data inside it can vanish. This command creates a folder named qdrant_storage right where you are standing in your terminal and links it to Qdrant's internal storage. Your vector embeddings will safely persist on your computer's hard drive even if you turn off the container.  
**--name qdrant-db**: Gives your container an easy-to-remember name instead of a random ID.  
qdrant/qdrant: The official image name on Docker Hub.

to verify if is running or not 
http://localhost:6333/dashboard



**Useful Docker Commands to Remember**  
Stop the database: **docker stop qdrant-db**   
Start it back up later: **docker start qdrant-d**   
Check running containers: **docker ps**

Delete it completely: **docker rm -f qdrant-db** (Your data folder qdrant_storage will still remain safe on your disk).


Handy commands:
docker ps                    # what's running
docker compose up -d         # start everything in the compose file
docker compose down          # stop and remove containers (mounted data stays)
docker system df             # how much disk Docker is using
docker system prune -a       # remove unused images/containers to free space




docker run -d -p 8080:80 --name x -e KEY=val image:tag   # start in background
docker ps / docker ps -a                                   # list containers
docker logs -f x                                           # view output
docker exec -it x bash                                     # get inside
docker stop x / docker start x                             # pause/resume
docker rm -f x                                             # delete
docker system prune                                        # remove all stopped containers & unused stuff


FROM python:3.13-slim        → take a box that already has Python 3.13
WORKDIR /app                 → inside the box, make a folder /app and go into it
COPY hello.py .              → copy hello.py from my Mac into /app
CMD ["python", "hello.py"]   → when the box starts, run: python hello.py



FROM image:tag        # starting point
WORKDIR /path         # create and cd into folder
COPY src dest         # Mac file → image
RUN command           # run at BUILD time (install stuff)
EXPOSE port           # document the port
CMD ["cmd", "arg"]    # run at START time
docker build -t name .     # build image from Dockerfile in this folder




