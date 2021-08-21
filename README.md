# go-graphql-hackernews

### Based in https://www.howtographql.com/graphql-go/ tutorial

1) Create a MySQL DB named `hackernews`
2) Change DB credentials with your own in `internal/pkg/db/migrations/mysql/mysql.go` and `server.go`
3) run `go install -tags 'mysql' github.com/golang-migrate/migrate/v4/cmd/migrate@latest   ` 
4) Run Migrations with the following command (*):
`migrate -database 'mysql://yourdbuser:yourdbpass@tcp(yourhost:yourport)/hackernews' -path internal/pkg/db/migrations/mysql up`
5) Run server `go run server.go`
6) In a browser, `localhost:8080` run some queries and mutations (GRAPHQL)


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

#### NOTE: Some Endpoints requires Authorization Token

1) Run this mutation in order to create an user:

```
mutation {
  createUser(input: {username: "user1", password: "123"})
}
```

2) Login in that user:

```
mutation {
  login(input: {username: "user3", password: "123"})
}
```

3) This will return the Auth Token

```
{
  "data": {
    "login": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE2MDE3MzIxMzgsInVzZXJuYW1lIjoidXNlcjMifQ.zMRRxQfwO7CpW58YVmtBVnlVvKQ3XbhF5B06VO4dFgI"
  }
}
```

4) Use this Token in bottom tab HTTP HEADERS, as follow:

```
{
  "Authorization": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE2MDE3MzIxMzgsInVzZXJuYW1lIjoidXNlcjMifQ.zMRRxQfwO7CpW58YVmtBVnlVvKQ3XbhF5B06VO4dFgI"
}
```


## Creating another tables or Modifying current

1) Modify `graph/schema.graphqls` with your own 
2) Run `go run github.com/99designs/gqlgen generate` (Assuming that you have gqlgen already. If not run first `go get github.com/99designs/gqlgen`)
3) Create your own migrations (*):
    1) `cd internal/pkg/db/migrations/`
    2) `migrate create -ext sql -dir mysql -seq create_yourmigration_table`
4) Create table in your own migrations:
    1) Example:
    ```
    CREATE TABLE IF NOT EXISTS Links(
        ID INT NOT NULL UNIQUE AUTO_INCREMENT,
        Title VARCHAR (255) ,
        Address VARCHAR (255) ,
        UserID INT ,
        FOREIGN KEY (UserID) REFERENCES Users(ID) ,
        PRIMARY KEY (ID)
    )
    ```
5) Run Migrations with the following command:
`migrate -database 'mysql://yourdbuser:yourdbpass@tcp(yourhost:yourport)/hackernews' -path internal/pkg/db/migrations/mysql up`
6) Create in internal a folder with your migration (i.e. links)
7) Create a go file inside, with your migration (i.e. `internal/links/links.go`)
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


## (*) Assuming that you have go-sql-driver and golang-migrate already. If not

1) Run `go get -u github.com/go-sql-driver/mysql`
2) Install golang-migrate
    1) `brew install golang-migrate` (MacOS) or `scoop install migrate` (Windows) or
    ```
        $ curl -L https://packagecloud.io/golang-migrate/migrate/gpgkey | apt-key add -
        $ echo "deb https://packagecloud.io/golang-migrate/migrate/ubuntu/ $(lsb_release -sc) main" > /etc/apt/sources.list.d/migrate.list
        $ apt-get update
        $ apt-get install -y migrate
    ``` 
    (Linux *debian based. I.E.: Ubuntu) 
3) Run `go build -tags 'mysql' -ldflags="-X main.Version=1.0.0" -o $GOPATH/bin/migrate github.com/golang-migrate/migrate/v4/cmd/migrate/` (If migrate doesn't exist in $GOPATH/bin/migrate, run go env, check your $GOPATH and create in it folder migrate)
