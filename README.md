#elasticsearch

This is a docker-compose configuration for ElasticSearch and eventually Kibana.

Official documentation on ElasticSearch in Docker

https://www.elastic.co/guide/en/elasticsearch/reference/6.7/docker.html

== Debugging

I went down a blind alley thinking that I needed to set up TLS security.
But I did learn a lot more about ES and how to change settings etc.

You can start bash to look at the default config settings like this:
```
  docker run -it elasticsearch:7.0.1 bash
```
Then you can look around and run bin/elasticsearch-* commands.
For example look at this default config file,

```
  [root@6ca5b51c0d1c elasticsearch]# cat config/elasticsearch.yml
  cluster.name: "docker-cluster"
  network.host: 0.0.0.0
```

And of course, you can do the same thing with kibana,
```
  docker run -it kibana:7.0.1 bash
```
You can find things in there like this:
```
  Visit [Elastic.co](http://www.elastic.co/guide/en/kibana/current/index.html) for the full Kibana documentation.
```
and this
```
bash-4.2$ cat config/kibana.yml
#
# ** THIS IS AN AUTO-GENERATED FILE **
#

# Default Kibana configuration for docker target
server.name: kibana
server.host: "0"
```

As you already know, running dockers like this will leave randomly named
unused images around which you will have to purge eventually.

== Start elasticsearch and kibana

The compose file will start them both up.
```
  docker-compose up
```

Test elasticsearch in the Docker server shell. If you run curl and get this, rejoice!

```
{
  % curl http://localhost:9200
  "name" : "es01",
  "cluster_name" : "docker-cluster",
  "cluster_uuid" : "WcK33arUS4u6_wu6-WYGZw",
  "version" : {
    "number" : "7.0.1",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "e4efcb5",
    "build_date" : "2019-04-29T12:56:03.145736Z",
    "build_snapshot" : false,
    "lucene_version" : "8.0.0",
    "minimum_wire_compatibility_version" : "6.7.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

If you are running a docker proxy then this or something similar should work,

  https://kibana.wildsong.biz

or if not, maybe this is good for you?

  http://localhost:5601/

=== Regarding TLS security

NB You don't need to do any of this. It is a trail of tears.
I am leaving some notes on certificates here for now because it cost me some time.

Elasticsearch uses plugins called "xpacks"
There appears to be a missing one. xpack-security.
It looks like it only matters if you are using a commercial version of elasticsearch anyway.

==== To create certificates

The instructions on elastic.co for creating certificates failed.

This generates a couple errors but still works.
One of them says i should be using elasticsearch-certutil instead of certgen

```
  mkdir certificates
  cp instances.yml certificates
  docker run -it -v `pwd`/certificates:/usr/share/elasticsearch/config/certificates elasticsearch:7.0.1 bash
 
  bin/elasticsearch-certgen --in config/certificates/instances.yml --out config/certificates/bundle.zip
  unzip config/certificates/bundle.zip -d config/certificates  
```
This puts certificates in a local folder, certificates, where docker-compose can access it.

