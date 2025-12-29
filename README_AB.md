# Teste de carga com Apache Bench em app React e em Docker

## Teste na máquina host (ubuntu)
atualizar o pacote apt
```
sudo apt-get update
```
instalar o apache
```
sudo apt-get install apache2-utils
```
Inicie a sua aplicação.

>*Caso você tenha realizado o passo a passo anterior( do ajuste de [redirecionamento](https://github.com/jv1nicius/Atividade1Nginx/blob/main/README_Redirecionamento.md) ):*
>>```
>>docker start loadbalancer node1 node2 node3 node4 node5
>>```
Realize o teste:

_Exemplo com um total de 1000 requisições, com no máximo 10 conexões simulâneas_
```
ab -n 1000 -c 10 http://localhost:84/
```
_Exemplo realizando com tempo ao invés de total de requisições e com 20 conexões simultâneas_
```
ab -t 15 -c 20 http://localhost:84/
```
## Teste com container para apache bench
Com a sua aplicação rodando, rode o seguinte código:
```
docker run -it --rm \ --add-host=host.docker.internal:host-gateway \
  httpd:2.4-alpine \
  ab -n 1000 -c 10 http://host.docker.internal:84/
```
O código acima utiliza uma imagem docker que possui o apache bench instalado para realizar o teste. Utilizamos também o `--add-host=host.docker.internal:host-gateway \` para que o container possa visualizar e acessar o localhost da máquina host, por fim temos o mesmo padrão de comando de teste `ab -n 1000 -c 10 http://host.docker.internal:84/`.
