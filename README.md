# Trabalho prático 1 de Segurança em Redes de Computadores

## Autores
- Ana Vidal (118408)
- Simão Andrade (118345)

## Estrutura do Relatório
- Introdução (ChatGPT)
- Estado-de-Arte (Simão)
- Rotas de Rede e conectividade (Ana)
- Load-Balancers (Simão e Ana)
- Configuração da Firewall:
  - Zonas e Regras (Ana e Simão);
- Questões Finais (Ana e Simão)
- Testes de Funcionamento (Ana e Simão)
- Conclusão (ChatGPT)

## Objetivo
Apresentar um relatório dos **testes de configuração** e de **funcionamento** dos cenários descritos nos pontos `9` e `10` do guia laboratorial “High-Availability Firewall Scenarios”.

Temos as seguintes tarefas a serem realizadas:

- [ ] Firewall and load-balancers deployment (2 valores).
- [ ] Network routing and connectivity (2 valores).
- [ ] Devices state synchronization (3 valores).
- [ ] Zones definition (3 valores).
- [ ] Inter-zone rules (6 valores).
- [ ] Report (4 valores).

## Estado-de-Arte

Explicar os seguintes conceitos:
- **Firewall**;
- **Zonas e Regras**;
- **Load Balancer**;
- **State Synchronization**;
- **Redundancy Synchronization**;

## Ponto 9

### Topologia

<p align="center">
  <img src="image.png" alt="Topologia" width="1000"/>
</p>

### Rotas e Conectividade da Rede

Vamos começar por atribuir os endereços IP às interfaces dos routers e aos computadores de acordo com o enunciado.

PC1 (computador interno):
```
ip 10.2.2.100/24 10.2.2.10
save
```

PC2 (computador externo):
```
ip 200.2.2.100/24 200.2.2.10
save
```

R1 (*router* interno):
```cli
conf t 
ip route 0.0.0.0 0.0.0.0 10.1.1.11
ip route 0.0.0.0 0.0.0.0 10.1.1.12
int f0/1
ip add 10.2.2.10 255.255.255.0
no shut
int f0/0
ip add 10.1.1.10/24
no shut
end
write
```

IP NAT: 192.1.0.0/23
Mascara do IP NAT: 255.255.254.0 

R2 (*router* externo):
```cli
conf t
ip route 192.1.0.0 255.255.254.0 200.1.1.11
ip route 192.1.0.0 255.255.254.0 200.1.1.11
int f0/1
ip add 200.2.2.10 255.255.255.0
no shut
int f0/0
ip add 200.1.1.10 255.255.255.0
no shut
end
write
```

LB1A (*load balancer* superior interno):
```cli
set system host-name LB1A
set interfaces ethernet eth0 address 10.1.1.11/24
set interfaces ethernet eth1 address 10.0.1.11/24
set interfaces ethernet eth2 address 10.0.6.1/24
set interfaces ethernet eth3 address 10.3.1.1/24
commit
save
```

LB1B (*load balancer* inferior interno):
```cli
set system host-name LB1B
set interfaces ethernet eth0 address 10.1.1.12/24
set interfaces ethernet eth1 address 10.0.5.1/24
set interfaces ethernet eth2 address 10.0.2.12/24
set interfaces ethernet eth3 address 10.3.1.2/24
commit
save
```

FW1 (*firewall* superior):
```cli
set system host-name FW1
set interfaces ethernet eth0 address 10.0.1.12/24
set interfaces ethernet eth1 address 10.0.5.2/24
set interfaces ethernet eth2 address 10.0.4.1/24
set interfaces ethernet eth3 address 10.0.7.1/24

set nat source outbound-interface eth0
set nat source rule 10 source address 10.0.0.0/8
set nat nat source rule 100 translation address 192.1.0.1-192.1.0.10

commit
save
```

FW2 (*firewall* inferior):
```cli
set system host-name FW2
set interfaces ethernet eth0 address 10.0.6.2/24
set interfaces ethernet eth1 address 10.0.2.13/24
set interfaces ethernet eth2 address 10.0.8.1/24
set interfaces ethernet eth3 address 10.0.3.1/24

set nat source outbound-interface eth0
set nat source rule 10 source address 10.0.0.0/8
set nat nat source rule 100 translation address 192.1.0.1-192.1.0.10

commit
save
```

LB2A (*load balancer* superior externo):
```cli
set system host-name LB2A
set interfaces ethernet eth0 address 200.1.1.11/24
set interfaces ethernet eth1 address 10.0.4.2/24
set interfaces ethernet eth2 address 10.0.8.2/24
set interfaces ethernet eth3 address 10.4.1.1/24

commit
save
```

LB2B (*load balancer* inferior externo):
```cli
set system host-name LB2B
set interfaces ethernet eth0 address 200.1.1.12/24
set interfaces ethernet eth1 address 10.0.7.2/24
set interfaces ethernet eth2 address 10.0.3.2/24
set interfaces ethernet eth3 address 10.4.1.2/24

commit
save
```

### Load-Balancers

LB1A (*load balancer* superior interno):
```cli
set load-balancing wan interface-health eth1 next-hop 10.0.1.12
set load-balancing wan interface-health eth2 next-hop 10.0.6.2
set load-balancing wan interface-health eth3 next-hop 10.3.1.2
set load-balancing wan rule 1 inbound-interface eth0
set load-balancing wan rule 1 eth1 weight 1
set load-balancing wan rule 1 eth2 weight 1
set load-balancing wan rule 1 eth3 weight 1
set load-balancing wan sticky-connections inbound
set load-balancing wan disable-source-nat

commit
save
```

LB1B (*load balancer* Superior externo):
```cli
set load-balancing wan interface-health eth1 next-hop 10.0.5.1
set load-balancing wan interface-health eth2 next-hop 10.0.2.13
set load-balancing wan interface-health eth3 next-hop 10.0.3.1
set load-balancing wan rule 1 inbound-interface eth0
set load-balancing wan rule 1 eth1 weight 1
set load-balancing wan rule 1 eth2 weight 1
set load-balancing wan rule 1 eth3 weight 1
set load-balancing wan sticky-connections inbound
set load-balancing wan disable-source-nat

commit
save
```

LB2A (*load balancer* inferior externo):
```cli
set load-balancing wan interface-health eth1 next-hop 10.0.4.1
set load-balancing wan interface-health eth2 next-hop 10.0.8.1
set load-balancing wan interface-health eth3 next-hop 10.4.1.2
set load-balancing wan rule 1 inbound-interface eth0
set load-balancing wan rule 1 eth1 weight 1
set load-balancing wan rule 1 eth2 weight 1
set load-balancing wan rule 1 eth3 weight 1
set load-balancing wan sticky-connections inbound
set load-balancing wan disable-source-nat

commit
save
```

LB2B (*load balancer* inferior interno):
```cli
set load-balancing wan interface-health eth1 next-hop 10.0.7.1
set load-balancing wan interface-health eth2 next-hop 10.0.3.1
set load-balancing wan interface-health eth3 next-hop 10.4.1.1
set load-balancing wan rule 1 inbound-interface eth0
set load-balancing wan rule 1 eth1 weight 1
set load-balancing wan rule 1 eth2 weight 1
set load-balancing wan rule 1 eth3 weight 1
set load-balancing wan sticky-connections inbound
set load-balancing wan disable-source-nat

commit
save
```

### Sincronização de Estados (*State Synchronization*)

### Definição de Zonas

### Regras entre Zonas

### Questões finais

1. Explain why the synchronization of the load-balancers allows the nonexistence of firewall synchronization.

R: A sincronização feita nos load balancers permite que os pedidos do cliente atinjam sempre o mesmo servidor, evitando que o firewall tenha de sincronizar estados entre os servidores.

Isto é feito através do conceito de *sticky sessions*, que permite que os pedidos do cliente sejam sempre encaminhados para o mesmo servidor, evitando que o firewall tenha de sincronizar estados entre os servidores.

2. Which load balancing algorithm may also allow the nonexistence of load-balancers synchronization?
3. Explain why device/connection states synchronization may be detrimental during a DDoS attack

## Ponto 10 (Ainda não chegámos aqui)
