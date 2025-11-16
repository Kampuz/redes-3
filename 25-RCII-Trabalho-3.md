# STP, RSTP e MSTP

Atividade 1 - Segundo Bimestre

Abigail Sayury Nakashima e Miguel de Campos Rodrigues Moret

Redes de Computadores II

2025

## 1.1 Introdução

Um loop de rede ocorre quando ocorre uma conexão entre duas portas de uma tomada de rede, essa ligação cria um loop de rede ethernet, esse tipo de ocorrência pode ser acidental ou intencional, em caso de necessidade de conferir falhas na infraestrutura da rede.

Este tipo de ocorrência deve ser feitas cautela pois esse tipo de ocorrência apresenta alguns problemas que afetam o desempenho da rede:

- **Broadcast storm**, onde quadros de broadcast seriam retransmitidos de forma excessiva na rede, pois os switches retransmitem quadros em broadcast através de suas demais portas, e quando há um caminho fechado, os quadros são retransmitidos continuamente pelos switches, inundando a rede, e consumindo a sua capacidade.
- **Instabilidade na tabela MAC nos switches**, cada switch tem uma tabela que mapeia endereços físicos MAC de dispositivos para as portas correspondentes em um switch, essa tabela então indica a porta por onde um quadro deve ser retransmitido para que seja transmitido para seu endereço MAC de destino. Como essa tabela é dinâmica, ela é constantemente atualizada e sobrescrita criando conflitos.
- **Múltiplos quadros repetidos recebidos em cada host**, cada quadro recebido também tem que ser processado, e isso cria um peso para o equipamento final no tempo de processamento desses quadros repetidos.

Afim de impedir que loops em redes façam de circular pacotes continuamente foi criado o protocolo STP, ou Protocolo Spanning Tree, que soluciona o problema de loops em redes LAN, conferindo a redundância na rede, bloqueando alguns caminhos garantindo que os dados se movam eficientemente.

Nas próximas seções desse trabalho serão tratados de forma mais aprofundada os protocolos STP, RSTP e MSTP, conceitos fundamentais para compreensão do funcionamento e entendimento dos protocolos, a arquitetura de rede proposta para a segunda parte do trabalho e o plano de endereçamento IP e VLANs definidas.

## 1.2 Protocolos STP, RSTP e MSTP

Como apresentado anteriormente, o protocolo STP foi desenvolvido para resolver problemas de redundância em redes Ethernet, sendo um protocolo padrão aberto e que impede broadcast storms, instabilidade da tabela MAC, loops e quadros múltiplos repetidos. Desde o seu desenvolvimento ocorreram atualizações: Rapid STP, ou RSTP, e Multiple STP, ou MSTP, que melhoraram o STP.

### 1.2.1 STP

O Spanning Tree Protocol foi definido inicialmente no padrão IEEE 802.1D. Foi o primeiro protocolo criado para evitar loops em redes Ethernet com caminhos redundantes. Seu funcionamento é baseado em uma árvore livre de ciclos, onde apenas um caminho lógico pode ser selecionado entre dois pontos de uma rede. Ele envolve quatro etapas principais:

- Eleger uma Root Bridge, que vai ser a raiz da árvore livre de ciclos.
- Encontrar topologias que possuem ciclos, enviando mensagens chamadas de BPDU, que, baseadas nelas, os switches encontram as partes com ciclos da topologia.
- Configurar os papéis das portas. Quando identificando um ciclo, cada switch coloca quantas portas de switch forem necessárias para que a topologia fique sem loops.
- Re-convergir em volta de falhas. Os switches continuam a trocar mensagens para acompanhar as conexões e a disponibilidade de switches adjacentes. Se um link cair, os switches executam os passos 2 e 3 novamente para ter certeza de que a nova topologia seja livre de ciclos.

### 1.2.2 RSTP

O Rapid STP foi criado pois o tempo para ser inicializado era muito grande, levando 50 segundos para um cliente se conectar para que pudesse então se comunicar em uma rede de dois switches conectados. Antigamente, esse tempo de até minutos era aceitável, mas atualmente isso é inaceitável.

A solução, que é o RSTP, foi que ele não depende de temporizadores para definir se deve colocar uma interface no estado de encaminhamento. Ao invés disso, o RSTP usa um handshake explícito para negociar o papel da porta e o estado, e esse é o mecanismo que faz o protocolo rápido. Essa versão converge mais rápido que o STP sem nenhuma configuração especial.

#### 1.2.2.1 O que torna o processo do protocolo rápido

É um processo de sincronização que aumenta a velocidade de convergência do RSTP comparado ao STP, e eles seguem os seguintes passos:

1. Quando uma porta de um switch (swi1) inicializa-se pela primeira vez, ela começa como uma porta designada no estado de bloqueio e começa a enviar BPDUs com um conjunto de bits de proposta, informando que esse switch quer se tornar a ponta designada para o segmento, incluindo o Bridge ID e o papel da porta.
2. Quando o outro switch (swi2) recebe o BPDU com a flag de proposta, ele começa um processo de sincronização, e é considerado sincronizado se estiver ou em estado de bloqueio ou se for uma porta de fronteira. Para alcançar esse estado, o switch temporariamente move todas as portas que não são de fronteira para o estado de bloqueio e checa se o BPDU recebido não está em conflito com os papéis das próprias portas.
3. Quando a checagem é completa, esse switch (swi2) seleciona um papel de porta e dá seguimento para colocar essa porta em estado de encaminhamento, o que então faz com que ela envie uma BPDU de volta para o swi1, com uma flag de consentimento.
4. Quando o swi1 recebe a BPDU com a flag de consentimento, ele sabe que a proposta foi aceita e move a porta designada para o estado de encaminhamento.

Esse processo de proposta e consentimento é muito rápido porque não depende de temporizadores STP, levando cerca de 0,3 segundos para começar o tráfego de encaminhamento.

#### 1.2.2.2 Papéis de Portas do RSTP

O RSTP seleciona o switch com o menor valor de prioridade de bridge como a root bridge e então define os seguintes papéis para cada porta:

- Porta root, o caminho de menor custo para alcançar a root bridge.
- Porta designada, a porta que conecta o switch com o melhor caminho para o segmento específico até a root bridge.
- Porta alternativa, o caminho de backup para a root bridge caso a porta atual falhe.
- Porta backup, a porta de backup para a porta designada.
- Porta desabilitada, porta que não participa na spanning tree.

#### 1.2.2.3 Estados das Portas do RSTP

**Discarding**: estado em que a porta descarta todos os frames que estão vindo e não aprende endereços MAC, combinando os estados Disabled, Blocking e Listening do STP.  
**Learning**: estado em que a porta ainda descarta todos os frames que estão vindo, mas começa a aprender endereços MAC.  
**Forwarding**: a porta encaminha frames baseado nos endereços MAC que já aprendeu e continua a aprender.

### 1.2.3 MSTP

O MSTP foi apresentado pelo padrão IEEE 802.1s, sendo baseado em seu antecessor RSTP, mas oferecendo a possibilidade de criar múltiplas árvores em uma única rede, melhorando a escalabilidade e a eficiência.

Cada árvore é uma instância, e cada instância pode ter sua própria root bridge e topologia de encaminhamento, assim como pode ser associada com uma ou mais VLANs, o que significa que diferentes VLANs podem seguir diferentes caminhos dentro de uma mesma rede física, sendo útil para redes maiores que precisam de maior flexibilidade e redundância.

#### 1.2.3.1 Implementação de um protocolo MSTP

A fim de ter implementado é necessário seguir os seguintes passos:

1. Definir a região MST, que é um grupo de switches que utilizam o MSTP e têm os mesmos parâmetros de configuração (nome, número de revisão e o mapeamento VLAN por instância), sendo crucial configurar para todos os switches regionais.
2. Configurar a instância MST (número, root bridge, papéis de porta).
3. Configurar a fronteira MST, que é o ponto em que a região se conecta a outra região ou a uma rede não MST, sendo necessário configurar para cada fronteira (CIST e CST).

#### 1.2.3.2 Vantagens e Desvantagens

O protocolo auxilia a reduzir instâncias de Spanning Trees e BPDUs, o que pode ajudar na redução do uso da CPU e memória nos switches, além de melhor utilizar links redundantes e balancear a carga, assim como simplificar a configuração da rede e oferecer compatibilidade com versões anteriores como STP e RSTP.

Entretanto, é necessário bom planejamento para evitar caminhos subótimos e ciclos, requerendo configuração consistente em todos os switches, e nem sempre suporta dispositivos legados ou de terceiros que não implementaram MSTP.

## 1.3 Conceitos Fundamentais

> *Explicar os **conceitos fundamentais**:*
> 
> - *Root Bridge*
> - *Port States (Blocking, Listening, Learning, Forwarding)*
> - *Bridge ID e Path Cost*
> - *Eleição e convergência da topologia*



## 1.4 Objetivos Gerais e Específicos

Os objetivos deste trabalho estão divididos em um objetivo geral e pbjetivos específicos, conforme descritos a seguir.

### 1.4.1 Objetivo Geral

Projetar e configurar uma rede corporativa de médio porte utilizando STP, RSTP e MSTP, analisando seu comportamento, tempos de convergência e desempenho em situações de falha.

### 1.4.2 Objetivos Específicos

- Compreender o funcionamento dos protocolos STP, RSTP e MSTP.
- Criar uma topologia de rede redundante em laboratório ou ambiente de produção.
- Implementar e comparar os três protocolos em cenários distintos ou iguais.
- Medir o tempo de convergência e estabilidade da rede (mesmo em redes simuladas).
- Propor boas práticas para ambientes corporativos com múltiplas VLANs (item não obrigatório).

## 1.5 Arquitetura de Rede Proposta

*Apresentar uma **arquitetura de rede proposta** com a quantidade de equipamentos (switches e computadores) e suas funções.*

*Descreva a topologia geral (quantidade de switches, hosts, servidores, etc.).*

*Indique funções dos dispositivos (core, distribuição, acesso).*

*Inclua um diagrama de rede (mesmo que conceitual).*

## 1.6 Endereçamento de IP e Esquema de VLANs

*Elaborar um **endereçamento IP e esquema de VLANs** adequado ao projeto. (item não obrigatório)*

*Mostre o plano de endereçamento IP e as VLANs definidas.*

*Explique a lógica de segmentação (por setor, função, departamento, etc.).*

*Pode incluir uma tabela com IPs, máscaras, VLAN IDs e descrições.*

https://www.pynetlabs.com/spanning-tree-protocol-explained/
https://www.tp-link.com/br/support/faq/3898/
https://glmtec.com.br/entenda-o-loop-de-rede-e-como-preveni-lo/
https://www.cisco.com/c/pt_br/support/docs/lan-switching/spanning-tree-protocol-stp-8021d/221722-troubleshoot-mac-flaps-loop-on-cisco-cat.html?utm_source=chatgpt.com
https://moodle3.ifsc.edu.br/mod/book/view.php?id=312208&chapterid=56881&lang=pt_br
https://www.networkacademy.io/ccna/spanning-tree/how-stp-works
https://www.networkacademy.io/ccna/spanning-tree/introduction-to-rapid-pvst
https://maisvch.com/pt/blog/qual-e-a-diferenca-entre-stp-mstp-pvst-e-rstp/
https://www.pynetlabs.com/what-is-mstp-protocol/
