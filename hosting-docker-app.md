# Build and push to registry
```
DOCKER_REGISTRY=docker.io
DOCKER_USER=arllanos
DOCKER_PASSWORD=<my_secret_password>
echo $DOCKER_PASSWORD |docker login $DOCKER_REGISTRY --username=arllanos --password-stdin
docker build -t $DOCKER_REGISTRY/$DOCKER_USER/minesweeper-api .
docker push $DOCKER_USER/minesweeper-api
```
# To run locally in docker compose
```
docker-compose up --build
docker-compose down --remove-orphans
```
# To run locally w/o docker compose but with a redis container
```
docker run --name redis -p 6379:6379 -d redis:alpine
go run main.go
```
# Azure hosted application
## Get credentials to connect to aks k8s cluster
`az aks get-credentials -g ResourceGroup1 -n minesweeper`

## Hosting details 
Github repo: https://github.com/arllanos/minesweeper-API
Application hosted in Azure AKS
	* Host: `191.233.1.78`
	* Port: `8080`
	* Example GET: `http://191.233.1.78:8080/games/game1/player1/board`
	
If using postman: https://documenter.getpostman.com/view/1621790/SzzgAKGJ
(Select "Azure" in the environment dropdown, or "Local" if running locally)
