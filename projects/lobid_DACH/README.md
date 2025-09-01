
# lobid organisations

This project is configured to download(well, partly)/prepare/build a complete Pelias installation for https://lobid.org/organisations.

See https://github.com/hbz/lobid-organisations/issues/419.

Atm not all queries result accurate data, so there is room for proper configuration and data input.

# Setup
Change to the proper directory:
```bash
$ cd /opt/pelias/project/lobid_DACH/
```
Please refer to the instructions at https://github.com/pelias/docker in order to install and configure your docker environment.

The minimum configuration required in order to run this project are [installing prerequisites](https://github.com/pelias/docker#prerequisites), [install the pelias command](https://github.com/pelias/docker#installing-the-pelias-command) and [configure the environment](https://github.com/pelias/docker#configure-environment).

Please ensure that's all working fine before continuing.

## Data directories

- `/opt/pelias/projects/lobid_DACH/data` for elasticsearch data (not compatible with NFS)
- `/data/pelias` NFS mount for backup of temporary download files, see https://github.com/pelias/docker?tab=readme-ov-file#optionally-cleanup-temporary-files 

## Elasticsearch
Tweak elasticsearch to use 8GB:
```bash
$ docker exec -u 0 -it pelias_elasticsearch bash
$ vi config/jvm.options # here change the -Xmx and -Xms to 8GB
$ pelias elasticsearch stop; pelias elasticsearch start; # restart elasticseach
```

# Run a Build

To run a complete build, execute the following commands:

```bash
pelias compose down # if it was up before
pelias compose pull
pelias elastic start
pelias elastic wait
pelias elastic create
pelias download all
pelias prepare all
pelias import all
pelias compose up
pelias compose ps # check all instances
pelias test run
```

# Make an Example Query

Residing in the hbz subnet, you can now make queries against this Pelias build.
The following query is an example resulting not very accurate data:

* @curl "http://gaia.hbz-nrw.de:4000/v1/search?layers=address&text=Juelicherstrasse%206,%2050674%20Koeln" |jq .@

Inaccurate data can be identified when `confidence` is equal or less than `0.6`.

In contrast, the following query results accurate geo data:

* @curl "http://gaia.hbz-nrw.de:4000/v1/search?layers=address&text=Gleueler%20Str.%2050%2050931%20Koeln" |jq .@

See that `confidence` has a value of `0.8`.
