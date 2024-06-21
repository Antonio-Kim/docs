## Adding Database

We'll implement two different database as an example here: using Sqlite and PostgreSQL. The code should have minimal difference
in any application in terms of coding but PostgreSql requires little more update.

### Before Database setup

Some prior installation of packages are required in order to use database. We'll use ORM approach here, which is EF core.

```
dotnet tool install --global dotnet-ef
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet add package <database ORM>
```

The last part depends on which database to use. Here are some of the package for each database:

- PostgreSQL: Npgsql.EntityFrameworkCore.PostgreSQL
- SQLite: Microsoft.EntityFrameworkCore.Sqlite
- Microsoft Server: Microsoft.EntityFrameworkCore.SqlServer

Initial setup is complete. Now let's move on to the databse specific setup

### PostgreSQL

This database requires little more configuration due to the nature of requiring to login and setup. But once that's passed the
coding is about the same. Using pgAdmin, open up and start up the server you want to create the database. Under "Login/Group
Roles", right click and select "Create -> Login/Group Role". Under general enter in the username, and under definition enter
in the password. Go to privileges,make sure to enable create database and login, or simply enable the superuser. Click save.

Now right click "Databases" and hit "Create". Enter in the name of the database and assign the new user you created as the
owner. To test whether you have access to the database or not, go to your terminal and type in the following:

```bash
psql -h localhost -d <databse> -U <username>
```

it will then ask for password which you created above. But if you're me and you don't want to deal with GUI, here's how you would
do it using command lines:

```bash
psql -U <superuser>
CREATE ROLE <new_user> WITH LOGIN PASSWORD='<password>';
ALTER ROLE <new_user> [Privileges(example: "CREATEDB", "LOGIN")]
CREATE DATABASE [IF NOT EXIST] db_name OWNER <new_user>;
```

this is only setting up the user and creating a database. Other commands such as configuring schema and others is not in this
documentation.

#### Side Note: Setting up with Docker

Here's an example of how to setup postgres with docker container. Eventually it will be moved to Docker compose, and then perhaps
into Kubernetes but for now, it's a single container only. We're using Alpine version for smaller size. Note that even alpine
verison has psql installed.

```bash
docker run --name test-database -e POSTGRES_PASSWORD=postgres -d postgres:alpine
docker exec -it test-postgres sh
psql -U postgres
```

(This does not ask for password when logging in as postgres user. Will need to look into why this might be the case).
