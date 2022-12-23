
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

- Note: While working with SQL DB in docker image for the very first time, 

## Branches:
- sql-image: The DBs are hosted in `mcr.microsoft.com/mssql/server:2019-latest` container; mounted in local filesystem
- `TODO:` az-sql: The DBs are hosted in Azure SQL DBs

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

- Issue 6: Azure SQL DB connection issue
    > microservice-compose-api.discount-1  | Unhandled exception. Microsoft.Data.SqlClient.SqlException (0x80131904): A network-related or instance-specific error occurred while establishing a connection to SQL Server. The server was not found or was not accessible. Verify that the instance name is correct and that SQL Server is configured to allow remote connections. (provider: TCP Provider, error: 35 - An internal exception was caught)  
    > microservice-compose-api.discount-1  |  ---> System.Net.Internals.SocketExceptionFactory+ExtendedSocketException (00000005, 0xFFFDFFFF): Name or service not known
    - Add correct client ip address in firewall rule for Azure SQL Server
    - Check SQL Server Host Name - do not prefit it with `tcp`, etc

- Issue 7: Error while `docker-compose up` - `Error response from daemon: network _key_ not found`  
    - https://stackoverflow.com/questions/39640963/error-response-from-daemon-network-myapp-not-found/69266310#69266310
    - Solution: Remove container (not Image) and then run again

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

-   > microservice-compose-ui.web-1              | warn: Microsoft.AspNetCore.DataProtection.Repositories.FileSystemXmlRepository[60]  
    > microservice-compose-ui.web-1              |       Storing keys in a directory '/root/.aspnet/DataProtection-Keys' that may not be persisted outside of the container. > Protected data will be unavailable when container is destroyed.  
    > microservice-compose-ui.web-1              | warn: Microsoft.AspNetCore.DataProtection.KeyManagement.XmlKeyManager[35]  
    > microservice-compose-ui.web-1              |       No XML encryptor configured. Key {489d93a9-8dae-4f77-898e-ba1443daf1fd} may be persisted to storage in unencrypted form.
