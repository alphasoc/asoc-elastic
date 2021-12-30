# asoc-elastic

This project provides instructions (this README) for configuring an end-to-end network
threat monitoring solution based on parts of the [Elastic](https://www.elastic.co) stack,
and on AlphaSOCs [Network Flight Recorder](https://github.com/alphasoc/nfr), Analytics
Engine (AE) and Console.  [Packetbeat](https://www.elastic.co/beats/packetbeat) is used
to capture and analyze network packets.  This data is then sent to
[Elasticsearch](https://www.elastic.co/elasticsearch/) for indexing.  NFR queries
Elasticsearch, extracting relevant network telemetry, and sends this data to AE, which
analyzes it for potential network threats.  Any such detections will be visible, and
actionable, in the AlphaSOC Console.

At the time of writing this guide, Elastic v7.16.2, and NFR v1.11.1 were used.

For details about AE, visit:
[AlphaSOCs Analytics Engine](https://docs.alphasoc.com/ae/architecture/).

**Note** that the instructions provided here are for a local installation, targeted at
evaluating the use of Elastic alongside AlphaSOCs AE and Console.  Integration with
the cloud-based Elasticsearch Service is not covered in this document.

## Architecture

**TODO** plantuml diag?
packetbeat sucks in pkts
NFR sucks in telemetry

    host                                                      net
    -------------------------------------------------         -------------------------
           pkts           processing                                          Console
    intf  ------> packetbeat -->  elastic (indexes)                              ^
    promisc.                       ^                 delivers                    |
                                   |--> NFR -------------------> AE (analyzes) --+
    -------------------------------------------------         -------------------------

## Who is this for?

If you're an existing AlphaSOC customer and would like to try NFR + Elastic as a network
monitoring solution, this is for you.

If you're not a customer, but are curious about the kinds of threats that AlphaSOC can help
you detect on your network, this is for you as well.

## AlphaSOC Registration

Since this guide focuses on setting up NFR + Elastic to work with AE, an AlphaSOC account
and API key are needed to get started.  If you don't have an account, please visit
[Demo | AlphaSOC](https://alphasoc.com/demo/) and register for one.  The registration
will grant you a **free**, 30 day demo of AlphaSOC services.  You'll receive an email
with a verification link.  Be sure to verify the account before continuing.

Once you have an account, please visit: [Console | AlphaSOC](https://console.alphasoc.net)
and sign in with your credentials.  Then, head over to https://console.alphasoc.net/credentials
and in the _API Keys_ pane, create a new API key by clicking _+ New API Key_, setting a
description for the key if you wish, and finally clicking _Generate_.  **Be sure to copy
the API key at this point to your clipboard for later use!  You will not be able to copy
it later on and will need to generate a new key.**

**TODO**: screens of the above?

**NOTE:** If you're an AlphaSOC developer using the staging variant of AlphaSOC services,
see the [Developers](#developers) section before continuing on.

## Installation

This section will cover local installation of Elasticsearch, Packetbeat and NFR.

### Elasticsearch

Installation instructions can be found [here](https://www.elastic.co/downloads/elasticsearch).
You can choose to download a pre-built package for your platform, or use a package repository.

### Packetbeat

In a similar fashion, Packetbeat installation instructions can be found
[here](https://www.elastic.co/downloads/beats/packetbeat).  Again, you can choose to download
a pre-built package, or use a package repository.

### NFR

To install NFR, visit the [releases](https://github.com/alphasoc/nfr/releases) page on GitHub.
There you will find Debian and RPM packages, as well as zipped archives (tarballs) containing
pre-built NFR binaries for Debian and Centos based Linux distributions, and for Windows.
Download an appropriate package.

If you downloaded a `.deb` or `.rpm`, use your system's package management utility to install
the package.  As part of installation, an NFR configuration file will be installed to
`/etc/nfr/config.yml`.

If you downloaded a Linux or Windows tarball, start by creating a directory where NFR will reside.
For example, under your home directory, create an `nfr` folder/directory.  Untar the contents
of the NFR tarball to this new `nfr` directory.  Finally, download the sample
[config.yml](https://raw.githubusercontent.com/alphasoc/nfr/master/config.yml) and copy it to
the `nfr` directory.

If a package is not available for your system, or if you want to build NFR from source, you will
need a Go compiler.  Go installation is covered [here](https://go.dev/doc/install).  Assuming
you have Go installed, you can clone the NFR Git [repositry](https://github.com/alphasoc/nfr.git),
change to the repository root directory, and build NFR.  Note that a `config.yml` will be present
in the repository root directory.

### TODO
- no NFR config.yml in binary tarballs!
- ./nfr account register -> creates a config.yml, also registers a new user and provides an API
  key; but not for existing users... maybe keep the section about AlphaSOC Registration as it's
  uniform?

## Configuration

This section will cover configuration of Elasticsearch, Packetbeat and NFR.  To help with clarity,
the following assumptions will be made:

1. `~/` represents your user home directory
1. `$ELASTIC_VER` represents the Elastic version
1. Elasticsearch has been installed locally to `~/elasticsearch-$ELASTIC_VER`
1. Packetbeat has been installed locally to `~/packetbeat-$ELASTIC_VER`
1. NFR has been installed locally to `~/nfr`, and a vanilla `config.yml` is present

If this is not the case for you, please augment the instructions in this section to fit your
installation locations.

### Packetbeat



### Elasticsearch

### NFR

## Developers

**TODO**




# Notes
// List indicies
$ curl localhost:9200/_cat/indices?v
// yellow open   packetbeat-7.16.2-2021.12.23-000001 XgHLM2xaS-iRpVzBxd7NOw   1   1       2239            0      1.2mb          1.2mb


curl -XGET --header 'Content-Type: application/json' http://localhost:9200/packetbeat-7.16.2-2021.12.23-000001/_search -d '{ "docvalue_fields": [ { "field": "@timestamp", "format": "strict_date_time" } ], "_source": [ "source.ip", "source.port", "destination.ip", "destination.port", "network.protocol", "destination.bytes", "source.bytes" ], "size": 10000, "query": { "bool": { "must": [ { "exists": { "field": "@timestamp" } }, { "exists": { "field": "source.ip" } }, { "exists": { "field": "destination.ip" } } ] } } }'

curl -XGET --header 'Content-Type: application/json' http://localhost:9200/packetbeat-7.16.2-2021.12.23-000001/_search -d '{ "docvalue_fields": [ { "field": "@timestamp", "format": "strict_date_time" }, { "field": "event.kind" } ], "_source": [ "source.ip", "source.port", "destination.ip", "destination.port", "network.protocol", "destination.bytes", "source.bytes" ], "size": 10000, "query": { "bool": { "must": [ { "exists": { "field": "@timestamp" } }, { "exists": { "field": "source.ip" } }, { "exists": { "field": "destination.ip" } }, { "exists": { "field": "event.kind" } } ] } } }'

curl -XGET --header 'Content-Type: application/json' http://localhost:9200/packetbeat-7.16.2-2021.12.23-000001/_search -d '{ "docvalue_fields": [ { "field": "@timestamp", "format": "strict_date_time" }, { "field": "event.created", "format": "strict_date_time" } ], "_source": [ "source.ip", "source.port", "destination.ip", "destination.port", "network.protocol", "destination.bytes", "source.bytes" ], "size": 10000, "query": { "bool": { "must": [ { "exists": { "field": "@timestamp" } }, { "exists": { "field": "source.ip" } }, { "exists": { "field": "destination.ip" } }, { "exists": { "field": "event.kind" } } ] } } }'

curl -XGET --header 'Content-Type: application/json' http://localhost:9200/packetbeat-7.16.2-2021.12.23-000001/_search -d '{ "docvalue_fields": [ { "field": "@timestamp", "format": "strict_date_time" } ], "_source": [ "source.ip", "source.port", "destination.ip", "destination.port", "network.protocol", "destination.bytes", "source.bytes" ], "size": 10000, "query": { "bool": { "must": [ { "exists": { "field": "@timestamp" } }, { "exists": { "field": "source.ip" } }, { "exists": { "field": "destination.ip" } } ], "filter": [ { "range": { "@timestamp": { "gte": "now-1m" }}}] } } }'

curl -XGET --header 'Content-Type: application/json' http://localhost:9200/packetbeat-7.16.2-2021.12.23-000001/_search -d '{ "docvalue_fields": [ { "field": "@timestamp", "format": "strict_date_time" }, {"field": "event" } ], "_source": [ "source.ip", "source.port", "destination.ip", "destination.port", "network.protocol", "destination.bytes", "source.bytes" ], "size": 10000, "query": { "bool": { "must": [ { "exists": { "field": "@timestamp" } }, { "exists": { "field": "source.ip" } }, { "exists": { "field": "destination.ip" } }, { "exists": { "field": "event" } } ], "filter": [ { "range": { "@timestamp": { "gte": "now-1m" }}}] } } }'
