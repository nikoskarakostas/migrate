# migrate CLI
## Original Repo
https://github.com/golang-migrate/migrate/
## About this fork
I wanted to be able to run either sql migrations or execute a golang binary file doing changes to the database.
## Example usage
Below is an example of a .go file of a migration.
#### example file: [id]_add_ten_example_tables.up.go
```go
package main // must be main

import (
    "database/sql"
    "fmt"
)

var (
	CONN         *sql.Conn  // mandatory
	DB           *sql.DB    // mandatory
	TargetSchema *string    // mandatory
	ERR          *error     // mandatory
)

func Migration() { // must be Migration [this gets called]
    
    // We do whatever we want here

    // for example,
	for i := 0; i < 10; i++ {
		_, err := DB.Exec(fmt.Sprintf(`
			CREATE TABLE %s.example_%d (
				id INT(6) UNSIGNED AUTO_INCREMENT PRIMARY KEY,
				field1 VARCHAR(30) NOT NULL,
				field2 VARCHAR(30) NOT NULL
			);
		`, *TargetSchema, i))   // You must use the TargetSchema in order to target it. Otherwise, the default schema is targeted.
		if err != nil {
			*ERR = err          // This is how we catch errors
			return              // This is how we catch errors
		}
	}

}
```
#### Build this and add to your migrations dir:
```bash
go build -buildmode=plugin -o /your/migrations/directory[id]_add_ten_example_tables.up.so [id]_add_ten_example_tables.up.go
```
#### notes:
| Databases supported | Sources supported |
|---------------------|-------------------|
| mysql               | file              |
|                     | s3                |


- File extension of the built file must be .so
- Changes must be contained in the Migration function. This is the one that will be executed. Follow the structure of the example above.
- CONN, DB, ERR declarations are mandatory. These are accessed by the package and get to point to package's memory addresses.

## Basic Functionality 
In a nutshell, we read the bytes of the binary file, fetch them, create a temporary file, write the feched data to it and call the [plugin](https://golang.org/pkg/plugin/) package in order to execute the binary's Migration func.