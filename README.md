# URL Shortener 

## Functional Requirements 
1. Users should be able to submit a long url and receive a shortened version of it
- Optionally users should be able to specify a custom alias for their URL 
- Optionally users should be able to specify an expiration date for their shortened url
2. Users should be able to access the original url using the shortened version 

## Core Entities 
<img width="681" alt="image" src="https://github.com/user-attachments/assets/cf1a1da2-ffc0-4575-89a3-cf6b7702d6ed">


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
<img width="478" alt="image" src="https://github.com/user-attachments/assets/2a4383a4-c50a-4de5-8810-a652840868f1">

### Users should be able to submit a long url and receive a shortened version of it
1. Client submits a short url by making a `POST` request to `/urls` and optionally providing an alias and expiration_date
2. Main server receives the request and validates whether the url is a valid url and it does not exist in the db
- url validation can be done using a library
- url existence can be checked using a DB Query
```
SELECT EXISTS (
    SELECT 1
    FROM long_url
    WHERE url = :original_url
) AS record_exists;
```
3. Upon successful validation
- If the user has not provided an alias, the server generates it
- If the user has not provided an expiration date, the server generates It
- If the `original_url` does not exist in db, a new record is created in both `original_url` and `short_url`
- If it does exist, a new record is created in `short_url` pointing to corresponding record in `original_url`
4. Upon successful creation, alias is returned back to client

### Users should be able to access the original url using the shortened version 
1. Client submits a `GET` Request to `/urls/{:alias}`
2. Server extract alias from request and queries db to find corresponding original url
```
SELECT ou.original_url
FROM original_url AS ou
JOIN short_url as su ON su.original_url_id=ou.id
WHERE su.alias=:alias
```
- If url does not exist -> `HTTP 404` is returned to client
- If url has expired -> `HTTP 410` is returned to client
otherwise -> `HTTP Redirect 302` is returned to client

### Scaling considerations
<img width="743" alt="image" src="https://github.com/user-attachments/assets/f2769e0d-69d7-49e6-9536-0cc8c7f1b16d">
- `api-gateway` is introduced which will be responcible for request routing, rate limit or any authentication. 
- To ensure availability and horizontal scaling, multiple replicas of services and the db can be created. The `url-redirection-service` is likely to have more replicas since the system is expected to have a significantly larger number of read compared to write operations. 
- Redis can be used to store a global counter used when generating the url alias. Lua script can be used for the increment operation to be atomic given the system is distributed.
- Redis cache is used for caching `alias: original_url` to optimize read operations.

