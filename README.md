# Dependency-Track Project Local Setup with modifications
This guide will briefly describe how to install and make changes to the Dependency-Track source code. The idea behind the changes is to add functionality to Dependency-Track to allow for a custom vulnerability source.

## Software Installation

1. Install **Chocolatey**: 
    ```powershell
    Set-ExecutionPolicy Bypass -Scope Process -Force
    [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072
    iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
    ```
   
2. Install **Git**:
    ```powershell
    choco install git
    ```
    
3. Install **NodeJS**, **npm** and **http-server**:
    ```powershell
    choco install nodejs
    node -v
    npm -v
    npm install -g http-server
    ```

4. Install **Docker** following the instructions on the [official website](https://docs.docker.com/get-docker/).

## Configuration of Local Dependency-Track
1. Clone the DepTrack Repositories:
    This repositories is required for customization of DepTrack functionality and adding new features

    ```powershell
    git clone https://github.com/DependencyTrack/dependency-track.git
    git clone https://github.com/DependencyTrack/frontend.git
    ```

2. Downloads the latest Docker Compose file
    ```powershell
    curl -LO https://dependencytrack.org/docker-compose.yml
    ```
3. Starts the stack using Docker Compose
    ```powershell
    docker-compose up -d
    ```

## Local Dependency-Track with custom changes
1. In general you need to make changes in the source code of DepTrack (frontend part), add your functionality and then rebuild the project, and create your own image, according to these steps, but it does not work correctly, at this stage and can create problems with this custom image! To try this method you can do the following steps:

    ### Make you changes in such files:
    - ..\frontend\src\views\administration\Administration.vue
    - Add new VulnSourceCustomSource.vue file into ..\frontend\src\views\administration\vuln-sources
    - ..\frontend\src\views\administration\AdminMenu.vue

    ### Then you need to rebuild your project and create **dist** folder for frontend part
    ```powershell
    cd "C:\Users\Volodymyr_Kumurzhy\OneDrive - EPAM\EPMUASP\frontend"
    npm install
    npm run build
    ```

    ### Once the project is built, you can build a Docker image:
    ```powershell
    docker build -f docker\Dockerfile.alpine -t dependencytrack/frontend-local .
    ```

    ### Then you have to update your **docker-compose.yml** and use your locally created image
    ```yml
      dtrack-frontend:
        image: dependencytrack/frontend-local #usage of new image
    ```

    ### Then you should just run new docker-compose file, ut for unknown reasons there is an error that there is no necessary script in the image to start the container

2. Another option for testing local changes, at least in the frontend part, is to use the http server that was installed earlier. To do this you need:

    ### Make you changes in such files:
    - ..\frontend\src\views\administration\Administration.vue
    - Add new VulnSourceCustomSource.vue file into ..\frontend\src\views\administration\vuln-sources
    - ..\frontend\src\views\administration\AdminMenu.vue

    ### Starts the stack using Docker Compose with default images and stop & remove frontend container
    ```powershell
    docker-compose up -d
    docker stop ***-dtrack-frontend-1
    docker rm ***-dtrack-frontend-1
    ```

    ### You need to start an http server with requests redirected to the deptrack api server port, for this you need to use the command:
    ```powershell
    $env:VUE_APP_SERVER_URL="http://localhost:8081" ; npm run serve
    ```

    ### After these steps, Deptrack will run on localhost 8080 and redirect requests to the server side of the application to localhotst 8081.