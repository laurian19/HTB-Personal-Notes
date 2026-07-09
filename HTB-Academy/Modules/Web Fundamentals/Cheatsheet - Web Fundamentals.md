
---
# URL Structure

```
scheme://user:password@host:port/path?param=value#fragment 
``` 

Example: 

``` 
https://admin:password@example.com:443/dashboard.php?user=admin&id=1#profile 
```

| **Component**  | **Example**          | **Description**                                                                                                                                                                       |
| -------------- | -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Scheme`       | `http://` `https://` | This is used to identify the protocol being accessed by the client, and ends with a colon and a double slash (`://`)                                                                  |
| `User Info`    | `admin:password@`    | This is an optional component that contains the credentials (separated by a colon `:`) used to authenticate to the host, and is separated from the host with an at sign (`@`)         |
| `Host`         | `inlanefreight.com`  | The host signifies the resource location. This can be a hostname or an IP address                                                                                                     |
| `Port`         | `:80`                | The `Port` is separated from the `Host` by a colon (`:`). If no port is specified, `http` schemes default to port `80` and `https` default to port `443`                              |
| `Path`         | `/dashboard.php`     | This points to the resource being accessed, which can be a file or a folder. If there is no path specified, the server returns the default index (e.g. `index.html`).                 |
| `Query String` | `?login=true`        | The query string starts with a question mark (`?`), and consists of a parameter (e.g. `login`) and a value (e.g. `true`). Multiple parameters can be separated by an ampersand (`&`). |
| `Fragments`    | `#status`            | Fragments are processed by the browsers on the client-side to locate sections within the primary resource (e.g. a header or section on the page).                                     |

**To Remember** 
- `/` → usually returns `index.html` 
- Multiple query parameters use `&` 
- Fragment (`#...`) is **never sent to the server**

--- 
# HTTP Flow 

``` Browser │ ▼ DNS lookup │ ▼ Server IP │ ▼ HTTP Request │ ▼ HTTP Response ``` 

Browser checks: ``` /etc/hosts ``` before asking DNS.

--- 
# cURL Essentials 
### View webpage 

``` 
curl http://example.com 
``` 
### Download file 

```
curl -O http://example.com/file.txt 
``` 
### Download with custom filename 

```
curl -o local.txt http://example.com/file.txt 
``` 
### Silent mode 

``` 
curl -s URL 
``` 
### Ignore invalid SSL certificate 

```
curl -k https://example.com
``` 
### Verbose request + response 

```
curl -v URL 
``` 
### Show response headers only 

```
curl -I URL 
``` 
### Include headers + body 

```
curl -i URL 
``` 
### Basic authentication 

```
curl -u user:password URL 
``` 

or 

```
curl http://user:password@example.com 
``` 
### Set User-Agent 

```
curl -A "Mozilla/5.0" URL 
``` 
### Add custom header 

```
curl -H "Header: Value" URL 
``` 
### POST data 

```
curl -X POST -d "username=admin&password=admin" URL 
``` 
### JSON POST 

```
curl -X POST \ -H "Content-Type: application/json" \ -d '{"key":"value"}' URL 
``` 
### Send cookie 

```
curl -b "PHPSESSID=abc123" 
``` 
### Follow redirects 

```
curl -L URL 
``` 

--- 
# Useful Flags 

| Flag | Meaning                    |
| ---- | -------------------------- |
| `-v` | Verbose                    |
| `-i` | Headers + body             |
| `-I` | Headers only (HEAD)        |
| `-H` | Add header                 |
| `-A` | User-Agent                 |
| `-u` | Basic auth                 |
| `-d` | POST data                  |
| `-X` | HTTP method                |
| `-b` | Cookie                     |
| `-L` | Follow redirects           |
| `-k` | Ignore SSL errors          |
| `-s` | Silent                     |
| `-O` | Save using remote filename |
| `-o` | Save using chosen filename |

---
# Default Ports 

| Protocol | Port |     |
| -------- | ---- | --- |
| HTTP     | 80   |     |
| HTTPS    | 443  |     |

---
# Quick Tips 

- `GET` → parameters in URL 
- `POST` → parameters in body 
- Browser DevTools → **Network** tab shows requests 
- Right click request → **Copy as cURL** 
- `Authorization: Basic ...` = Base64(username:password) 
- `Authorization: Bearer ...` = JWT authentication and includes a longer encrypted value
- Cookies usually maintain authenticated sessions

---
# Request Methods

|**Method**|**Description**|
|---|---|
|`GET`|Requests a specific resource. Additional data can be passed to the server via query strings in the URL (e.g. `?param=value`).|
|`POST`|Sends data to the server. It can handle multiple types of input, such as text, PDFs, and other forms of binary data. This data is appended in the request body present after the headers. The POST method is commonly used when sending information (e.g. forms/logins) or uploading data to a website, such as images or documents.|
|`HEAD`|Requests the headers that would be returned if a GET request was made to the server. It doesn't return the request body and is usually made to check the response length before downloading resources.|
|`PUT`|Creates new resources on the server. Allowing this method without proper controls can lead to uploading malicious resources.|
|`DELETE`|Deletes an existing resource on the webserver. If not properly secured, can lead to Denial of Service (DoS) by deleting critical files on the web server.|
|`OPTIONS`|Returns information about the server, such as the methods accepted by it.|
|`PATCH`|Applies partial modifications to the resource at the specified location.|

---
# Status Codes 

|**Class**|**Description**|
|---|---|
|`1xx`|Provides information and does not affect the processing of the request.|
|`2xx`|Returned when a request succeeds.|
|`3xx`|Returned when the server redirects the client.|
|`4xx`|Signifies improper requests `from the client`. For example, requesting a resource that doesn't exist or requesting a bad format.|
|`5xx`|Returned when there is some problem `with the HTTP server` itself.|

---
# CRUD

| Operation | HTTP Method | Description                                        |
| --------- | ----------- | -------------------------------------------------- |
| `Create`  | `POST`      | Adds the specified data to the database table      |
| `Read`    | `GET`       | Reads the specified entity from the database table |
| `Update`  | `PUT`       | Updates the data of the specified database table   |
| `Delete`  | `DELETE`    | Removes the specified row from the database table  |

---
