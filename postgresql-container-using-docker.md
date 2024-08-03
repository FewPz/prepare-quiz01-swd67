## 📚 - PostgreSQL container using Docker

To set up a PostgreSQL container using Docker with a specific username and password, you can follow these steps:

Pull PostgreSQL Docker Image: Pull the latest PostgreSQL image from Docker Hub using the following command
```sh
docker pull postgres
```

Run PostgreSQL Container: Run the PostgreSQL container with environment variables to set the username and password. Use the -e option to set environment variables.
```sh
docker run
  --name my_postgres
  -e POSTGRES_USER=myusername
  -e POSTGRES_PASSWORD=mypassword
  -e POSTGRES_DB=mydatabase
  -p 5432:5432
  -d postgres
```
Here’s a breakdown of the command:
* `--name my_postgres`: Names the container my_postgres.
* `-e POSTGRES_USER=myusername`: Sets the PostgreSQL username.
* `-e POSTGRES_PASSWORD=mypassword`: Sets the PostgreSQL password.
* `-e POSTGRES_DB=mydatabase`: (Optional) Creates a new database named mydatabase.
* `-p 5432:5432`: Maps the container’s port 5432 to your host machine’s port 5432.
* `-d`: Runs the container in detached mode. 
* `postgres`: Specifies the Docker image to use.

Verify the Container is Running: Check if the container is running using the following command:
```sh
docker ps
```
* You should see your my_postgres container listed with its status as "Up".

Access the PostgreSQL Database You can access the PostgreSQL database using psql or any PostgreSQL client tool with the credentials you specified.
```sh
psql -h localhost -U myusername -d mydatabase
```
It will prompt you for the password you set (mypassword).
