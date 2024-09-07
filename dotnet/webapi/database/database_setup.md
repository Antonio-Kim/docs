## Adding Database

We'll setup various databases here for webapi to use. The code should have minimal difference in any application in terms of coding but many SQL DBMS will requires little more configuration.

### Before Database setup

Some prior installation of packages are required in order to use database. We'll use ORM approach here, which is EF core.

```bash
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

This database requires little more configuration than Sqlite for example due to the nature of requiring to login and setup. But once that's passed the
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

## Scaffolding Models using existing database

There are two approaches to use EF Core:

1. Database first: a database exists so you build model that matches its structure and features
2. Code first: No database exists, so you build a model and then use EF core to create a database that matches its structure and features

The database first approach requires reverse engineering the database, a method called scaffolding. This section will briefly focus on the scaffolding.

Assuming that you have a database with table and dataset, we can export that database into model using EF Core. Here's the code using SQL server and explanation afterwards:

```bash
dotnet ef dbcontext scaffold "Data Source=tcp:127.0.0.1,1433;Initial Catalog=Northwind;Integrated Security=true;TrustServerCertificate=true;" Microsoft.EntityFrameworkCore.SqlServer --output-dir Models --namespace Northwind.Console.EFCore.Models --data-annotations --context NorthwindDb
```

`dotnet ef dbcontext scaffold` is the main command here, and then the source of the database. There may be a case where you cannot use Integrated Security. If so, then you'll need to replace the Integrated Security with SQL server Authentication:

```bash
"Data Source=tcp:127.0.0.1,1433;Initial Catalog=Northwind;User ID=your_username;Password=your_password;TrustServerCertificate=true;"
```

### Mapping Inheritance hierarchies with EF Core

There are three strategies dealing with inheritance in EF Core: **table-per-hierarchy (TPH)**, **table-per-type (TPT)**, and **table-per-concrete-type (TPC)**. We'll take the following example as to how to handle hierarachical dataset

```csharp
public abstract class Person {
	public int Id { get; set; }
	public string? Name { get; set; }
}

public class Student : Person {
	public string? Subject { get; set;}
}

public class Employee : Person {
	public DateTime HireDate { get; set; }
}
```

#### TPH strategy

TPH will use single table with a single column that includes "discriminator" to differentiate which type of person - Student or Employee.

| Id  | Name        | Discriminator | Subject | HireDate   |
| --- | ----------- | ------------- | ------- | ---------- |
| 1   | Roman Roy   | Student       | History | NULL       |
| 2   | Kendall Roy | Employee      | NULL    | 02/04/2014 |
| 3   | Siobhan Roy | Employee      | NULL    | 12/09/2020 |

The benefit of this is its simplicity and performance, and usually the default strategy by EF Core.

#### TPT Mapping strategy

TPT uses a table for every type. The benefit is that it reduces stroage, but disvantage is that single entity is spread over multiple table, which reduces overall performance. Thus TPT is a a poor choice and will not bt discussed any further.

#### TPC strategy

TPC uses a table for each non-abstract type, and the benfit is performance because when querying a single concrete type, only one table is needed so we avoid expensive join. It works best for large inheritance hierarchies of many concrete types, each with many type-specific properties.

Students Table

| Id  | Name      | Subject |
| --- | --------- | ------- |
| 1   | Roman Roy | History |

Employees Table

| Id  | Name        | Hire Date  |
| --- | ----------- | ---------- |
| 2   | Kendall Roy | 02/04/2014 |
| 3   | Siobhan Roy | 12/09/2020 |

#### Configuring each strategies

Both strategies require setting up tables for each entity:

```csharp
public DbSet<Person> People { get; set; }
public DbSet<Student> Students { get; set; }
public DbSet<Employee> Employees { get; set;}
```

You are finished for TPH. For TPC you'll need to state the following: `modelBuilder.Entity<Person>().UseTpcMappingStrategy();` You can then optionally name each table

```csharp
modelBuilder.Entity<Student>().ToTable("Students");
modelBuilder.Entity<Employee>().ToTable("Employees");
```

TPC strategy must have shared sequence, and we configure it as following:

```csharp
modelBuilder.HasSequence<int>("PersonIds");

modelBuilder.Entity<Person>().UseTpcMappingStrategy()
	.Property(e => e.Id).HasDefaultValueSql("NEXT VALUE FOR [PersonalIds]");
```
