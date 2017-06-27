# nginx example for rewriting based on request body content

This example uses nginx via docker. The `content_by_lua` directive is used to read the request body and match on it.

## Running

```bash
docker run -it --rm -v `pwd`/default.conf:/etc/nginx/conf.d/default.conf -v `pwd`/nginx.conf:/nginx/conf/nginx.conf -v `pwd`/static:/usr/share/nginx/html -p 8080:8080 danday74/nginx-lua
```

## Requests

The `./static` folder has 2 files: `abc.json` and `xyz.json`.
These can be retrieved statically by doing either of the following

```bash
curl localhost:8080/abc.json
{
  "value": "abc"
}

curl localhost:8080/xyz.json
{
  "value": "xyz"
}
```

However, they can _also_ be retrieved by sending a `POST` request to the `/example` endpoint with a json request body that includes a key `file` with a value of either `abc` or `xyz`.

```bash
curl localhost:8080/example -d '{"file":"abc"}'
{
  "value": "abc"
}

curl localhost:8080/example -d '{"file":"xyz"}'
{
  "value": "xyz"
}
```

## How does it work?

The nginx config file with these rules is at `./default.conf`. Here is the lua logic for reading the request body, matching on it, and serving content based on the match.

```
        content_by_lua '
            ngx.req.read_body()

            if ngx.re.match(ngx.var.request_body, "\\\"abc\\\"") then 
                ngx.exec("/abc.json")
            elseif ngx.re.match(ngx.var.request_body, "\\\"xyz\\\"") then 
                ngx.exec("/xyz.json")
            else
                ngx.status = 404
                ngx.exit(ngx.OK)
            end 
        ';
```
