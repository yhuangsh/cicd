# dev container: 

- same image standard for all developers: 
- each developer maps his own git work copy to a standard container path /project and run as root there
- developer test their code locally before commit/push. 
- Jenkins also uses this container to pull code, build, test, make release, lastly create deployment image. Everything runs within this container
- Dockerfile used to create this dev container is in the cicd project.
- Building of this image is also automated by the same cicd process 

# release container:
docker build -t 