
## Motivation:

To deploy existing `.Net Core 3.1` microservices as `docker` containers; using `docker-compose` (manually created) spinning all microservices up.

## Prerequisites

- Docker Desktop 4.14.1 (91661)
- Visual Studio 2019/2022

## Intructions

- Start Docker Desktop
- Execute below command from commands prompt from the root directory

    ```console
    docker-compose build
    docker-compose up
    ```
    Once done, clear the containers with below command

    ```console
    docker-compose down
    ```

- **Note:**  
While working with SQL DB in `docker image` for the very first time, you may have to do `docker-compose up` at-least twice.  
The reason is: the required DBs for microservices have not yet initialized; which in turn won't start the dependant microservices.  
Doing this will get the DBs created sequentially (by EF - check the docker logs and your mounted folder.)  
Once all the DBs get created, all the microservices will start.

## Branches:

- sql-image: The DBs are hosted in `mcr.microsoft.com/mssql/server:2019-latest` container; mounted in local filesystem

## Dev Notes:

### Issues encountered:

- Issue 1: `Failed to compute cache key: ".csproj" not found` while running docker-compose build, 
    - https://stackoverflow.com/questions/66933949/failed-to-compute-cache-key-csproj-not-found
    - Solution - use `build` object instead of `build dockerfileParentFolder`
        ``` lang: python
        build:
           context: .
           dockerfile: GloboTicket.Services.Discount/Dockerfile
        ```

- Issue 2: `...then an error occurred during the pre-login handshake.`
    - https://github.com/dotnet/SqlClient/issues/1479#issuecomment-1015587779
    - Solution: set `TrustServerCertificate=True` in your connection string
    
- Issue 3: `Discount MS` URL showing can't reach this page `http://localhost:5007/api/discount/code/Awesome`
    - Updated "applicationUrl": "http://localhost:5007" in launchSettings.json (https -> http)
    - Now URL shows proper response: http://localhost:5007/api/discount/code/Awesome

- Issue 4: Can't reach sql.discount DB from SSMS (outside of api.discount docker container)

- Issue 5: Font files not getting loaded  
    > Failed to load resource: the server responded with a status of 404 (Not Found)     OpenSans-Bold.ttf:1  
    > Failed to load resource: the server responded with a status of 404 (Not Found)     OpenSans-Regular.ttf:1  

    - The file names are case-sensitive in linux based OS/docker-images. Updating the names properly in CSS fixes this issue.

### Done:
- Use mssql server container for database and mount volumn
- Start other services with DB dependencies and start using Az Message Bus
- Provide config details from compose env file
- Fixed errors in Payment & Marketing microservices

### TODO:
- Use docker-compose.override.yml
- Configure application to use https: Reference: https://medium.com/@woeterman_94/docker-in-visual-studio-unable-to-configure-https-endpoint-f95727187f5f
- Use JWT and OpenAuth
- Error at the time of placing the order `Something went wrong placing your order. Please try again.`