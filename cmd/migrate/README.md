# migrate CLI
## Original Repo
https://github.com/golang-migrate/migrate/
## About this fork
I wanted to be able to run either sql migrations or execute a golang binary file doing changes to the database. Below is an example of a .go file of a migration.
## Example usage

#### example file: [id]_add_ten_example_tables.up.go
```go
package main

import (
	"database/sql"
	"fmt"
)

var (
	CONN *sql.Conn
	DB   *sql.DB
	ERR  *error
)

func Migration() {

	for i := 0; i < 10; i++ {
		_, err := DB.Exec(fmt.Sprintf(`
			CREATE TABLE example_%d (
				id INT(6) UNSIGNED AUTO_INCREMENT PRIMARY KEY,
				field1 VARCHAR(30) NOT NULL,
				field2 VARCHAR(30) NOT NULL
			);
		`, i))
		if err != nil {
			*ERR = err
			return
		}
	}

}
```
#### Build this and add to your migrations dir:
```bash
go build -buildmode=plugin -o /your/migrations/directory[id]_add_ten_example_tables.up.so [id]_add_ten_example_tables.up.go
```
#### notes:
- Souces supported: s3, file
- Databases supported: mysql

- File extension of the built file must be .so
- Changes must be contained in the Migration function. This is the one that will be executed. Follow the structure of the example above.
- CONN, DB, ERR declarations are mandatory. These are accessed by the package and get to point to package's memory addresses.