//Video tutorials:
	https://www.youtube.com/watch?v=f0lMGPB10bM
	https://www.youtube.com/watch?v=r_tGl4zF1ZQ

         --------------------------- How to create Docker Image and Container from it ---------------------------

//1. Create a project
//2. Navigate to the Folder which contains .csproj and there cteate a new file "Dockerfile" with Uppercase D and without extension. DockerFile - that is a text document(list with instructions) which tell to Docker how to build an Image
//3. Within the Docker file write list with commands. Something like this
//////////////////////////////////////////////////////////////////////////////////////////////////////////////
		FROM mcr.microsoft.com/dotnet/sdk:5.0 AS build-env 
		WORKDIR /app

		COPY *.csproj ./
		RUN dotnet restore

		COPY . ./
		RUN dotnet publish -c Release -o out

		FROM mcr.microsoft.com/dotnet/aspnet:5.0
		WORKDIR /app
		EXPOSE 80
		COPY --from=build-env /app/out .
		ENTRYPOINT ["dotnet", "coffee.dll"]
//////////////////////////////////////////////////////////////////////////////////////////////////////////////
Explanation
	FROM mcr.microsoft.com/dotnet/sdk:5.0 AS build-env --> go to hub.docker.com and pull Image with name mcr.microsoft.com/dotnet/sdk:5.0 and give it a name build-env
	WORKDIR /app --> this create and set the Working directory in the Linux container(which will became Image)
	COPY *.csproj ./ --> Copy all files with .csproj extension from actual PC into working container
	RUN dotnet restore --> start dotnet with parameter restore. That pull all dependencies for all projects from Nuget/Npm hub.
	COPY . ./ --> now copy the rest of files from actual PC into working container
	RUN dotnet publish -c Release -o out --> Here we run dotnet publish with configuration flag Release, which will build app for deployment in output directory (flag -o) with name out
	FROM mcr.microsoft.com/dotnet/aspnet:5.0 --> go to hub.docker.com and pull Image with name mcr.microsoft.com/dotnet/aspnet:5.0
	EXPOSE 80 --> Open this contnainer at port 80
	COPY --from=build-env /app/out . --> copy from build-env, folder /app/out everything in current directory
	ENTRYPOINT ["dotnet", "coffee.dll"] --> Here we specify how to Run our app. We will start dotnet and when it start after that run coffee.dll file and that will start application

//4. Install Docker for Desktop --> After that we can write docker command in commad prompt
//5. Create Image from DockerFile --> docker build [-t || --tag] [image_name]:[image_TAG] [folder that contains Dockerfile]. 
//How to create a custom Image - Remember Images should contains everything that our app needs to run - all dependecies, all source code ... etc
"docker build --tag coffee_image:latest ."//This will create new Image with name = coffee_image and version latest (a.k. TAG) and don't forget dot after latest. This dot specify where is our Dockerfile. In our case is only . because we are in F:\projects\coffee and in this current dir we have Dockerfile. If we are in other dir we must apply correct path to Dockerfile
	Examples:		
		docker build --tag start:v1 .
		docker build -t    start:v1 F:\Projects\startbootstrap-freelancer-master
//6. Create new container
"docker run --name coffeeContainer -d -p 1122:80 coffee_image:latest" //Create new container with name=coffeeContainer from Image with name=coffee_image and TAG=latest. That container can be open from browser at localhost:1122 as port 112 will be mapped to port 80 at a container
//7. Now go to the browser and enter this address localhost:1122 and see result!

//8. How to upload my Image in hub.docker.com and Share with other people
	//1. Register in hub.docker.com and take DockerID and password
	//2. Open command prompt and enter command --> docker login
	//3. enter username and password
	//4. Now we have to bind/tag/map Image with DockerID - with this command
	// docker tag [Image_name]:[Image_TAG] [DockerID]/[the name of the new Image in hub.docker.com]
	// example --> docker tag coffee_image:latest 8301125369/firstimage
	// If we run <--docker images --> command will see a new Image there with name 8301125369/firstimage. This is the Image that will be push to the hub
	//5. now push Image in the hub with command --> docker push 8301125369/firstimage
	//6. Go to hub.docker.com and will see new Image 8301125369/firstimage there

	

Additional commands:
docker image ls //show all images
docker ps //show only running containers (it is equal to this command "docker container ls")
docker ps -a // show all containers, running and stopped 

docker start website //Starts the container with name website
docker stop website //Stop the container with name website --> docker stop [name of the container] || [Id of the container]

docker rm website // rm - stands for remove
docker rm -f website //By default it is not possible to remove running container. It must be stopped first and then removed. But with the flag '-f' - that means 'force' it is possible to remove conatiner without first to stop it.
docker rmi coffee_image // remove Image with name coffee_image

docker image inspect coffee_image // This command inspect image with name coffee_image. Give as additional information


//how to format the list with containers view
docker ps --format="ID\t{{.ID}}\nNAME\t{{.Names}}\nImage\t{{.Image}}\nPORTS\t{{.Ports}}\nCOMMAND\t{{.Command}}\nCREATED\t{{.CreatedAt}}\nSTATUS\t{{.Status}}\n"

//How to get in the conatiner to see all files in it
F:\Projects\website>docker exec -it website bash // That say ---> 'Docker run Linux command prompt(bash) within container with name website '. And result is =>
root@2f10cfa673c8:/# ---> Here we are inside container with runing linux bash in root directory
//We must to navigate to /usr/share/nginx/html folder to see out index.html file. How to do that@
cd /usr
cd share
cd nginx
cd html
/usr/share/nginx/html# ls - ('ls' is equal to DIR)
//It's better to use ls -l --> that is long listing command. That will show detail info like DIR command do.

cd ../ - go to parent directory
cd ../../ - go two times to parent directory
cd -  ---> go to previous directory  example: root/users/share/nginx/html after typing this command cd ../../ports - we will go to root/users/share/ports. When we intering cd - will jump from  root/users/share/ports to root/users/share/nginx/html (something as a callStack)

//How to pull Image from hub.docker.com
docker pull [Image_name] // we take the Image name from hub.docker.com

//Volume in docker
docker run --name website -v  F:\Projects\website:/usr/share/nginx/html -d -p 3333:80 nginx:latest - That create container with --name 'website', also map all files and dir wich are in this folder F:\Projects\website to container's folder /usr/share/nginx/html a.k. share them from PC to container and vice versa, -d stands for 'Detached mode' -p for 'port mapping from 3333 in browser to port 80 in server' nginx:latest - stands for It will be created new container (instance) of this Image. 

//Share data between containers
docker run --name website2 --volumes-from website -d -p 5555:80  nginx:latest //This command create new container with name website2 and take all volume data from container website and put it within new created conatiner. When we start localhost:5555 will see exactly same thing as when we start localhost:3333 (that is the address when works container one)
