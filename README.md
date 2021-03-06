## Job interview assignment completed...

## Prerequisites:

* [docker-ce](https://docs.docker.com/engine/installation/)
* [docker-compose](https://docs.docker.com/compose/install/)

Alternatively please check the `docker_install.sh` script (has been tested on the *Debian Stretch* and *Jessie*).


## Initial setup:

Make sure there's no app on your host currently bound to the tcp port *8888* or *5432*.

Change to the `ratestask_sre` directory and run the following command:

```
$ cd ratestask_sre
$ docker-compose up --build
```

It will build `ratestask_app` and `ratestask_db` containers and start them for you. 

Once the containers are running, please navigate your browser to:

* http://127.0.0.1:8888/get - for the GET request
* http://127.0.0.1:8888/put - for the POST request


#### Example POST:

```
$ curl -sX POST -d date_from=2016-01-09 -d date_to=2016-01-19 -d origin_code=CNGGZ -d destination_code=eetll -d price=1234 http://127.0.0.1:8888/put
{
  "success": true
}
```

#### Example GET:
```
$ curl -s 'http://127.0.0.1:8888/get?date_from=2016-01-10&date_to=2016-01-11&origin=cnggz&destination=eetll'
[
  {
    "average_price": "1187",
    "day": "2016-01-10"
  },
  {
    "average_price": "1186",
    "day": "2016-01-11"
  }
]
```

Currently benchmarks on the *localhost* show ~252 req/s. Such stats in my opinion are totally acceptable.

### Notes:

Currently `prices` table allow duplicate records, which perhaps is a design flaw. I would rather alter
the table schema and add the `primary key` based on the *orig_code*, *dest_code* and *day* columns:

```
ALTER TABLE prices ADD PRIMARY KEY (orig_code, dest_code, day);
```

This way INSERT query could be changed to the following and benefit from the *UPSERT* (UPdate or inSERT):

```
INSERT INTO prices (orig_code, dest_code, day, price)
VALUES %s ON CONFLICT (orig_code, dest_code, day) DO UPDATE
SET (orig_code, dest_code, day) = '(EXCLUDED.orig_code, EXCLUDED.dest_code, EXCLUDED.day);
```
