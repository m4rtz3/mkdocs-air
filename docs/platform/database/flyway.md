
## Flyway

Flyway is an open-source database migration tool that strongly favors simplicity and convention over configuration. It is designed to simplify the process of versioning a database, similar to how Git versions source code. 

With Flyway, you can apply version control to your database which allows you to migrate it to a newer version and also revert changes if needed. Flyway uses SQL scripts or Java-based migrations to evolve your database schema in a way that is controllable and predictable.

Key features of Flyway include:

- **Version control for your database**: Allows you to track changes and apply version control to your database, similar to how you would with your source code.
- **Support for SQL and Java-based migrations**: You can use SQL for simple changes, and Java for complex migrations.
- **Repeatable migrations**: You can use this feature to manage objects in your database that can't be easily handled with versioned migrations, like stored procedures and views.
- **Multiple database support**: Flyway supports a wide variety of databases including MySQL, PostgreSQL, SQL Server, and more.

[^1]: https://www.baeldung.com/database-migrations-with-flyway


https://www.baeldung.com/liquibase-vs-flyway