# fictitious.iocs
[![Banner](banner.png)]()
> Aqui eu apresento um arquivo de log do apache ```access.log``` que sofreu diversos ataques da web, o objetivo deste projeto é fornecer um log com diversos tipos de IOCs (Indicator Of Compromise) para treinamento de tecnicas SIEM.  
> Neste arquivo de logs você encontrará ataques do tipo SQLi, XSS, Brute-Force, Directory parser, LFI, RCE e muito mais !


# Criando ambiente para instalação com multipass:
> O objetivo desta seção é utilizar o snap para instalar o multipass para que possamos obter uma vm ubuntu rapida e facil para implementar o fictitious.iocs.  

## Instalando o Snap e Multipass:  
```
$ apt-get update
$ apt-get install snapd
$ sudo snap install multipass
$ sudo ln /snap/bin/multipass /usr/bin/multipass
$ sudo multipass --help

```

## Criando VM ubuntu com Multipass:  
```
$ sudo multipass launch --name SIEM --disk 20G --mem 6G
$ sudo multipass list
$ sudo multipass info SIEM
$ sudo multipass shell SIEM
```

### Instale o oracle java:  
> https://www.oracle.com/technetwork/java/javase/downloads/jdk10-downloads-4416644.html  
#### Terminal:  
```
$ sudo apt-get update
$ sudo apt-get install -y default-jre
$ java --version
```  

## Baixando e configurando a pilha ELK:  
> Acesse https://elastic.co/ e baixe o elasticsearch, kibana e logstash.  
#### Terminal (elasticsearch e kibana):  
> A configuranção de terminal não é necessario baixar os arquivos do site elastic.co pelo browser.  

### ELASTICSEARCH  
```
$ wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.9.2-linux-x86_64.tar.gz
$ tar zxvf elasticsearch-7.9.2-linux-x86_64.tar.gz
$ cd elasticsearch-7.9.2

Execute o elasticsearch:
$ bin/elasticsearch
```  

### KIBANA  
```
$ sudo multipass shell SIEM
$ wget https://artifacts.elastic.co/downloads/kibana/kibana-7.9.2-linux-x86_64.tar.gz
$ tar zxvf kibana-7.9.2-linux-x86_64.tar.gz
$ cd kibana-7.9.2

Descomente e edite a linha 7 do arquivo config/kibana.yml:
$ vi config/kibana.yml
server.host: 0.0.0.0

Descomente a linha 28 do arquivo config/kibana.yml:
elasticsearch.hosts: ["http://localhost:9200"]

Execute o kibana:
$ bin/kibana
```

### LOGSTASH  
```
$ sudo multipass shell SIEM
$ wget https://artifacts.elastic.co/downloads/logstash/logstash-7.9.2.tar.gz
$ tar zxvf logstash-7.9.2.tar.gz
$ mkdir -p /home/ubuntu/SIEM/logs
$ cp fictitious.iocs/access.log /home/ubuntu/SIEM/logs/
$ cd logstash-7.9.2
$ sudo nano config/pipeline.conf

```  
#### pipeline.conf:  
```
input {
  file {
         path => "/home/ubuntu/SIEM/logs/access.log"
         start_position => "beginning"
         sincedb_path => "/dev/null"
  }
}

filter {
    grok { match => { "message" => "%{COMBINEDAPACHELOG}" } }

    mutate{ 
        convert => { "bytes" => "integer" } 
          }

    date { 
        match => [ "timestamp" , "yyyy/MM/dd HH:mm:ss" ] 
        target => "@timestamp"

         }

    geoip {  source => "clientip"    }
 
    useragent {

          source => "agent"
          target => "useragent"

              }
}

output {
  elasticsearch { hosts => ["localhost:9200"] }
}

``` 
#### Execute logstash:  
```
$ bin/logstash -f config/pipeline.conf --config.reload.automatic
```

#### Acesse http://localhost:5601  
> Discover >> Create index pattern >> Index pattern name = logstash-2020.xx.xx-000001 >> Next step >> Time field = @timestamp >> Create index pattern. Return Discover.  

## AppVuln_fictitious.iocs.tar.gz  
> Este arquivo contem as aplicações utilizadas para gerar o ```access.log```, neste tarball você poderá utilizar para treinar suas tecnicas de pentest tanto para blueteam quanto redteam.  
 
```
$ gunzip -d AppVuln_fictitious.iocs.tar.gz
$ tar xvf AppVuln_fictitious.iocs.tar
$ cd fictitious.iocs/bkp
$ ls -la
```  

:brazil:
