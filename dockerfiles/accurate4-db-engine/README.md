# Accurate 4 Database Engine

Configured Firebird database engine to serve Accurate 4.2 cient

Note: Tested and working on Accurate client version 4.2.13, Build: 1385 (Standard)

## Usage (server side)

```
docker run -d --name accurate4 -p 3051:3051 -v ~/accurate:/accurate suryastef/accurate4-db-engine
```

Put database file `*.gdb` inside `/home/(username)/accurate` directory

## Usage (client side)

Enter: Hostname/IP address then `/accurate/(database-name).gdb` when opening database in Accurate application