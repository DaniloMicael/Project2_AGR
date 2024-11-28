## BGP

1. Traffic from Coimbra should go preferably by Lisboa, while traffic from Aveiro should go preferably by Porto 
    - O tráfego que chega a Coimbra deve sair por lisboa. ou seja, a sair para a Internet, pra fora, deve ir por Lisboa
    - O tráfego que chega a Aveiro deve sair pelo Porto. ou seja, a sair para a Internet, pra fora, deve ir pelo Porto

2. National providers should not transport traffic between other national providers and the International provider 
    - O tráfego do National Provider 1 para a Internet deve ir direto para o International Provider, sem passar pelo National Provider 2
    - O tráfego do National Provider 2 para a Internet deve ir direto para o International Provider, sem passar pelo National Provider 1
    - Ou seja, National Providers não servem como trânsito para outros National Providers, cada um liga-se diretamente para fora

## MPLS

3. In  order  to  reduce  costs  and  have  a  better  security  control,  the  Sales  Department  of  CompanyA required a virtual private connection between the Aveiro and Lisboa poles, with a guaranteed bandwidth of  2Mbps. All  Videoconference  traffic  between  the Aveiro  and  Lisboa  poles  of  CompanyA  should  use this private connection. 
    - Configurar MPLS na rede do national provider1, no switch do centro, para fazer um tunel de aveiro pra lisboa, no caso do projeto é 2000 (nao 512 como no guiao), ou seja, usar o comando `ip rsvp bandwidth 2000 2000`

## IPSec

4. Due to security requirements, traffic between the Leiria and London networks should be encrypted
    - Tunel IPSec entre Leiria e Londres

## Other Notes

- É preciso BGP no National Provider 1 nos routers Porto, Lisboa, Coimbra e Aveiro
- É preciso BGP nos routers Coimbra e Aveiro para eu poder escolher por onde sair, ou seja, no router Coimbra configuro BGP para sair por Lisboa e no router Aveiro configuro para sair pelo Porto
- 