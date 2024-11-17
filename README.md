# URL Shortener 

## Functional Requirements 
1. Users should be able to submit a long url and receive a shortened version of it
- Optionally users should be able to specify a custom alias for their URL 
- Optionally users should be able to specify an expiration date for their shortened url
2. Users should be able to access the original url using the shortened version 

## Core Entities 

## Main Endpoints

#### Create a shortened url
- Method: `POST`
- URL: `/urls`
##### Request Headers Format
- Authorization: Bearer <JWT Token>
- The JWT token should contain `userId` and `password` claims.
##### Request Body Format
``` 
{
  "original_url": "string",       // The original URL to be shortened (required)
  "alias": "string",              // Custom alias for the shortened URL (optional)
  "expiration_date": "YYYY-MM-DD" // Expiration date for the shortened URL (optional)
}
``` 
##### Response Format
``` 
{
  "alias": "string" // The alias for the shortened URL (generated or custom)
}
```
##### Example Request
```
POST /urls HTTP/1.1
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Type: application/json

{
  "original_url": "https://example.com/long-url",
  "alias": "customAlias",
  "expiration_date": "2024-12-31"
}
```
##### Example Response
```
{
  "alias": "customAlias"
}
```

## HLD
### Users should be able to submit a long url and receive a shortened version of it
1. Client submits a short url by making a `POST` request to `/urls` and optionally providing an alias and expiration_date
2. Main server receives the request and validates whether the url is a valid url and it does not exist in the db
- url validation can be done using a library
- url existence can be checked using a DB Query
3. Upon successful validation
- If the user has not provided an alias, the server generates it
- If the user has not provided an expiration date, the server generates It
- If the `original_url` does not exist in db, a new record is created in both `original_url` and `short_url`
- If it does exist, a new record is created in `short_url` pointing to corresponding record in `original_url`
4. Upon successful creation, alias is returned back to client

### Users should be able to access the original url using the shortened version 
1. Client submits a `GET` Request to `/urls/{:alias}`
2. Server extract alias from request and queries db to find corresponding original url
- If url does not exist -> `HTTP 404` is returned to client
- If url has expired -> `HTTP 410` is returned to client
otherwise -> `HTTP Redirect 302` is returned to client

### Scaling considerations