# Estudo de Caso — Hospedagem de Servidor Minecraft na AWS EC2

**Área:** Infraestrutura em Nuvem / Suporte Técnico  
**Plataforma:** Amazon Web Services (AWS)  
**Duração estimada do projeto:** Contínuo (operação e manutenção)  
**Ambiente:** Windows Server em instância EC2 t2.micro (Free Tier)

---

## Contexto

Configurei e administrei um servidor privado de Minecraft hospedado na AWS para uma comunidade de até 30 jogadores. O objetivo era oferecer uma experiência estável e acessível, com o servidor disponível 24 horas por dia, sem depender de hardware local.

A escolha pela AWS se deu pela flexibilidade de configuração, controle total sobre o ambiente e possibilidade de operar dentro do nível gratuito (Free Tier) da plataforma.

---

## Ambiente Técnico

| Componente | Configuração |
|---|---|
| Provedor de nuvem | Amazon Web Services (AWS) |
| Serviço utilizado | EC2 (Elastic Compute Cloud) |
| Tipo de instância | t2.micro (1 vCPU, 1 GB RAM) |
| Sistema operacional | Windows Server |
| Aplicação | Servidor Minecraft (Java Edition) |
| Rede | Security Groups com liberação das portas 25565 (Minecraft) e 3389 (RDP) |
| Acesso remoto | Remote Desktop Protocol (RDP) |

---

## Problema Identificado

Após colocar o servidor em operação, os jogadores passaram a relatar quedas de conexão frequentes e aumento de latência (lag), especialmente durante horários de pico com múltiplos usuários conectados simultaneamente.

**Sintomas observados:**
- Desconexões intermitentes sem mensagem de erro clara
- Latência acima de 300ms em períodos de maior uso
- Lentidão na resposta do servidor mesmo com poucos jogadores online

---

## Diagnóstico

Para identificar a causa raiz, realizei as seguintes ações de investigação:

**1. Análise de recursos da instância**  
Acessei o servidor via RDP e verifiquei o Gerenciador de Tarefas. Identifiquei que o processo do Minecraft (`java.exe`) estava consumindo consistentemente entre 85% e 95% da CPU e ultrapassando o limite de RAM disponível, forçando uso de memória virtual (paginação em disco).

**2. Verificação dos logs do servidor**  
Analisei os arquivos de log do Minecraft (`latest.log`) e encontrei mensagens de "Can't keep up!" — indicador padrão de que o servidor não conseguia processar todos os ticks a tempo.

**3. Análise de rede**  
Verifiquei as configurações do Security Group na AWS e confirmei que as portas estavam corretamente abertas. O problema não era de conectividade externa, mas de desempenho interno da instância.

**Causa raiz identificada:** A instância t2.micro, com apenas 1 GB de RAM, era insuficiente para sustentar o processo Java do Minecraft com múltiplos jogadores simultâneos. Além disso, o servidor estava rodando com alocação de memória padrão (512 MB), abaixo do recomendado.

---

## Solução Implementada

**Ajuste 1 — Parâmetros de memória JVM**  
Editei o script de inicialização do servidor para alocar memória de forma explícita, alterando os parâmetros de inicialização do Java:

```
java -Xms512M -Xmx768M -jar server.jar nogui
```

Esse ajuste definiu um teto de memória compatível com o limite da instância, evitando paginação excessiva.

**Ajuste 2 — Otimização do server.properties**  
Reduzi a distância de renderização (view-distance) de 10 para 6 chunks e desativei funcionalidades desnecessárias para o perfil da comunidade, reduzindo a carga de processamento.

**Ajuste 3 — Agendamento de reinicialização automática**  
Configurei uma tarefa agendada no Windows Server (Task Scheduler) para reiniciar o processo do servidor diariamente em horário de baixo uso, liberando memória acumulada por vazamentos do Java.

**Ajuste 4 — Monitoramento de recursos**  
Ativei o monitoramento básico pelo CloudWatch da AWS para acompanhar o uso de CPU e identificar picos anormais com antecedência.

---

## Resultado

Após as mudanças, o servidor operou de forma estável com até 15 jogadores simultâneos, com:

- Redução das desconexões em aproximadamente 80%
- Latência estabilizada abaixo de 120ms na maioria das sessões
- Eliminação das mensagens de "Can't keep up!" nos logs
- Operação contínua sem intervenção manual por períodos superiores a 7 dias

O servidor permaneceu dentro do Free Tier da AWS, sem geração de custos adicionais.

---

## Aprendizados Técnicos

- **Dimensionamento de recursos em nuvem:** aprendi a correlacionar os requisitos de uma aplicação com os limites da instância antes de partir para o troubleshooting de rede.
- **Leitura de logs de aplicação:** o diagnóstico foi possível principalmente pela análise dos logs — prática diretamente aplicável ao suporte de sistemas.
- **Configuração de Security Groups:** entendi na prática como funciona o controle de tráfego de entrada e saída em ambientes AWS.
- **Acesso remoto via RDP:** configurei e utilizei o Remote Desktop para administração do servidor sem acesso físico à máquina.
- **Task Scheduler (Windows Server):** automatizei tarefas de manutenção preventiva sem necessidade de intervenção manual.

---

## Competências Demonstradas

`AWS EC2` · `Windows Server` · `RDP` · `Troubleshooting` · `Análise de logs` · `Otimização de performance` · `Automação de tarefas` · `Security Groups` · `CloudWatch` · `Gerenciamento de infraestrutura`

---

*Este projeto foi desenvolvido de forma independente como administrador de comunidade, sem vínculo empregatício formal.*
