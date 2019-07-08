# P4 INT - In-Band Network Telemetry

```
                   +--+
                   |h4|
                   ++-+
                    |
                    |
+--+      +--+     ++-+     +--+
|h1+------+s1+-----+s3+-----+h3|
+--+      +-++     +--+     +--+
            |
            |
          +-++
          |s2|
          +-++
            |
            |
          +-++
          |h2|
          +--+
```

## Introdução

Este exemplo usa o cabeçalho de opções do Ipv4 para armazenar estatísticas por salto, como id do switch, profundidade da fila e porta de saída. 

## Como executar

Configure o ambiente da Virtual Machine para execução conforme orientação no diretório [vm](./vm) 

Execute a topologia:

```
sudo p4run
```

Inicie o script do receptor em h2:

```
xterm h2
python receive.py "h2-eth0"
```

Enviar pacotes com o cabeçalho Ipv4 Options:

```
xterm h1
python send.py 10.0.2.2 "oi h2" 10
```

## Filebeat, Elasticsearch e Kibana

Para armazenar, indexar e gerar gráficos com dados coletados pelo INT, utilizamos o Filebeat, Elasticsearch e Kibana, o filebeat coletada os dados e enviar para indexar no elasticsearch, com o kibana podemos visualizar os dados, gerar gráficos e acompanhar em tempo real.

Para executar utilizamos containers Docker.
Partindo da premissa que já temos o Docker instalado, execute:

```
# start elasticsearch
docker run --name=elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:7.2.0
```

```
# start filebeat
docker run \
  --name=filebeat \
  --user=root \
  --volume="$(pwd)/filebeat.yml:/usr/share/filebeat/filebeat.yml:ro" \
  --volume="$(pwd)/log/packets.log:/var/log/packets.log:ro" \
  docker.elastic.co/beats/filebeat:6.5.4 filebeat -e -strict.perms=false
```


```
# start kibana
docker run --name=kibana --link elasticsearch -p 5601:5601 docker.elastic.co/kibana/kibana:7.2.0
```

