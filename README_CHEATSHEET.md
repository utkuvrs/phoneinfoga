# PhoneInfoga — Local Run Cheatsheet

Repo: Go backend (`go 1.20`) + Vue.js web client (`web/client`).

## 1. Prereqs

- Go >= 1.20
- (optional, only for web client dev/build) Node.js + yarn
- (optional) Docker

## 2. Fastest way: Docker (no local Go/Node needed)

```shell
# build image from this repo
docker build -t phoneinfoga .

# run web server + client (http://localhost:5000)
docker run --rm -it -p 5000:5000 phoneinfoga serve

# custom port
docker run --rm -it -p 8080:8080 phoneinfoga serve -p 8080

# REST API only, no web client
docker run --rm -it -p 5000:5000 phoneinfoga serve --no-client

# one-off scan
docker run --rm -it phoneinfoga scan -n "+33612345678"
```

Or use published image instead of building:
```shell
docker run --rm -it -p 5000:5000 sundowndev/phoneinfoga serve
```

## 3. Native Go run (backend only, client not embedded unless built first)

```shell
# from repo root
go run . scan -n "+33612345678"
go run . scanners
go run . serve -p 5000
go run . serve --no-client -p 5000
```

Notes:
- `serve` looks for a built client in `web/client/dist`. Without it, use `--no-client` (REST API only) or build client first (step 5).
- CLI flags: `-n/--number`, `-D/--disable <scanner>` (repeatable), `--plugin <path.so>` (repeatable), `--env-file <path>` (repeatable, default `.env`).

## 4. Build local binary via Makefile

```shell
make install-tools   # gotestsum, mockery, swag, golangci-lint
make build            # -> ./bin/phoneinfoga
./bin/phoneinfoga scan -n "+33612345678"
./bin/phoneinfoga serve
```

## 5. Web client dev (Vue.js, separate from Go server)

```shell
cd web/client
yarn install
yarn serve        # dev server w/ hot reload
yarn build        # -> web/client/dist (consumed by `phoneinfoga serve`)
```

## 6. Optional API keys (scanners disabled without them)

Put in `.env` at repo root (auto-loaded, or pass `--env-file`):

```
NUMVERIFY_API_KEY=xxxx
GOOGLE_API_KEY=xxxx
GOOGLECSE_CX=xxxx
GOOGLECSE_MAX_RESULTS=10   # optional, default 10, max 100
```

- `local` and `googlesearch` scanners need no keys (always run).
- `numverify` needs `NUMVERIFY_API_KEY`.
- `googlecse` needs `GOOGLE_API_KEY` + `GOOGLECSE_CX`.
- `ovh` needs no key but only runs for country codes 33/32/44/34/41.

## 7. Tests / lint

```shell
make test        # gotestsum, race, coverage
make coverage
make fmt
make lint         # needs golangci-lint (install-tools handles it)
```

## 8. Useful endpoints once `serve` running

```
GET  http://localhost:5000/api/
GET  http://localhost:5000/api/numbers/+33612345678/scan/local
GET  http://localhost:5000/api/numbers/+33612345678/scan/numverify
GET  http://localhost:5000/api/numbers/+33612345678/scan/googlesearch
GET  http://localhost:5000/api/numbers/+33612345678/scan/ovh
POST http://localhost:5000/api/v2/numbers   {"number":"+33612345678"}
```
Swagger: https://petstore.swagger.io/?url=https://raw.githubusercontent.com/sundowndev/phoneinfoga/master/web/docs/swagger.yaml
