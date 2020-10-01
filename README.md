# go-graphql-hackernews

### Based in https://www.howtographql.com/graphql-go/ tutorial

1) Create a MySQL DB named `hackernews`
2) Change DB credentials with your own in `internal/pkg/db/migrations/mysql/mysql.go` and `server.go`
3) Run Migrations with the following command:
`migrate -database 'mysql://yourdbuser:yourdbpass@tcp(yourhost:yourport)/hackernews' -path internal/pkg/db/migrations/mysql up`
4) Run server `go run server.go`
5) In a browser, `localhost:8080` run some queries and mutations (GRAPHQL)

#### GraphQL Query Example
```
query {
  links {
    title
    address
    id
  }
}
```

#### GraphQL Mutation Example
```
mutation create{
  createLink(input: {title: "StackOverflow", address: "https://stackoverflow.com/"}){
    title,
    address,
    id,
  }
}
```

## Creating another tables or Modifying current

1) Modify `graph/schema.graphqls` with your own 
2) Run `go run github.com/99designs/gqlgen generate` (Assuming that you have gqlgen already. If not run first `go get github.com/99designs/gqlgen`)
3) Create your own migrations (*):
    1) `cd internal/pkg/db/migrations/`
    2) `migrate create -ext sql -dir mysql -seq create_yourmigration_table`
4) Create table in your own migrations:
    a) Example:
    ```
    CREATE TABLE IF NOT EXISTS Users(
        ID INT NOT NULL UNIQUE AUTO_INCREMENT,
        Username VARCHAR (127) NOT NULL UNIQUE,
        Password VARCHAR (127) NOT NULL,
        PRIMARY KEY (ID)
    )
    ```
5) Run Migrations with the following command:
`migrate -database 'mysql://yourdbuser:yourdbpass@tcp(yourhost:yourport)/hackernews' -path internal/pkg/db/migrations/mysql up`
6) Create in internal a folder with your migration (i.e. users)
7) Create a go file inside, with your migration (i.e. `internal/users/users.go`)
8) In the created file, create your struct accord to GraphQL Schema. Check example in `internal/links/links.go`
9) Modify the resolver for your schema in `graph/schema.resolvers.go`

```
func (r *mutationResolver) CreateLink(ctx context.Context, input model.NewLink) (*model.Link, error) {
	var link links.Link
	link.Title = input.Title
	link.Address = input.Address
	linkID := link.Save()
	return &model.Link{ID: strconv.FormatInt(linkID, 10), Title:link.Title, Address:link.Address}, nil
}
```
10) Run server `go run server.go` and execute some mutations and queries


(*) Assuming that you have go-sql-driver and golang-migrate already. If not

1) Run `go get -u github.com/go-sql-driver/mysql`
2) Run `go build -tags 'mysql' -ldflags="-X main.Version=1.0.0" -o $GOPATH/bin/migrate github.com/golang-migrate/migrate/v4/cmd/migrate/` (If migrate doesn't exist in $GOPATH/bin/migrate, run go env, check your $GOPATH and create in it folder migrate)
