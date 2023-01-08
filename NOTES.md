# Uh oh, I don't know Go D:  

I'm not much of a linux guy, but I've tried running the minikube and such on Ubuntu running on WSL 2.  
One issue is that it's configured improperly, my default user is root, which minikube discourages, for good reason.  
  
I'm more familiar with DroneCI, and have my reservations about Github Actions. But I've chosen to use Github actions because of time constraints.  

I've also had a bad experience with minikube, we've had virtual machines that hardly ever worked. While docker-desktop seemed to work just fine.  
  
### Dockerfile considerations

Normally you'd use alpine, but because of needed ability to debug, a full image was chosen.  
In a prod-like setup, reducing privileges aswell as installed applications/tools is recommended, hence, alpine.  

The dockerfile is borrowed from https://hub.docker.com/_/golang  
  
If memory serves me correct, we used a kaniko-based approach at Nuuday for building images, the benefit was that the agents were not required to run as root.  

  
###