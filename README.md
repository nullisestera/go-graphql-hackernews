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
