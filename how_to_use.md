
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

