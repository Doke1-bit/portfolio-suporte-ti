# POP-001 — Reinicialização e Verificação de Instância EC2 Windows Server

**Tipo:** Procedimento Operacional Padrão (POP)  
**Código:** POP-001  
**Versão:** 1.0  
**Data de criação:** Abril 2026  
**Responsável:** Administrador de infraestrutura  
**Tempo estimado de execução:** 15–20 minutos

---

## 1. Objetivo

Descrever o procedimento padrão para reinicializar uma instância EC2 com Windows Server na AWS de forma controlada, garantindo que o ambiente seja verificado antes e após o processo para minimizar o impacto nos usuários e serviços em execução.

---

## 2. Aplicabilidade

Este procedimento deve ser seguido nas seguintes situações:

- Aplicação de atualizações de sistema operacional (Windows Update)
- Servidor apresentando lentidão persistente ou alto consumo de recursos
- Manutenção programada de infraestrutura
- Após alterações de configuração que exijam reinicialização

---

## 3. Responsabilidades

| Papel | Responsabilidade |
|---|---|
| Administrador de infraestrutura | Executar o procedimento e registrar ocorrências |
| Solicitante | Comunicar a necessidade e confirmar janela de manutenção |

---

## 4. Pré-requisitos

Antes de iniciar, confirme que todos os itens abaixo estão atendidos:

- [ ] Acesso ao Console AWS com permissão de gerenciamento de instâncias EC2
- [ ] Credenciais de acesso RDP disponíveis (usuário e senha)
- [ ] Arquivo `.pem` da instância disponível (caso necessário recuperar senha)
- [ ] Janela de manutenção comunicada aos usuários (se houver impacto)
- [ ] Backup recente dos dados críticos realizado (snapshot EBS recomendado)

---

## 5. Procedimento

### Fase 1 — Verificação prévia (antes da reinicialização)

**Passo 1 — Confirmar estado da instância no Console AWS**

1. Acesse o Console AWS em [https://console.aws.amazon.com](https://console.aws.amazon.com).
2. Navegue até **EC2 > Instâncias**.
3. Localize a instância pelo nome ou ID.
4. Confirme que o status é **Em execução** e que as verificações de status exibem **2/2 verificações aprovadas**.
5. Anote o **Endereço IPv4 público** atual para referência.

**Passo 2 — Verificar serviços em execução via RDP**

1. Conecte-se à instância via RDP.
2. Abra o **Gerenciador de Tarefas** (`Ctrl + Shift + Esc`) e anote o uso atual de CPU e memória.
3. Abra o **Gerenciador de Serviços** (`services.msc`) e verifique se os serviços críticos estão com status **Em execução**.
4. Salve ou anote qualquer processo em andamento que possa ser interrompido.

**Passo 3 — Notificar usuários (se aplicável)**

1. Comunique aos usuários ativos que o servidor será reinicializado.
2. Solicite que salvem seus trabalhos e encerrem as sessões ativas.
3. Aguarde a confirmação ou defina um tempo limite de espera (recomendado: 5 minutos).

---

### Fase 2 — Execução da reinicialização

**Passo 4 — Encerrar a sessão RDP**

1. Feche a conexão RDP de forma adequada (**Iniciar > Encerrar Sessão**).
2. Não utilize o botão "X" da janela RDP, pois isso desconecta sem encerrar a sessão no servidor.

**Passo 5 — Reinicializar via Console AWS**

1. No Console AWS, selecione a instância.
2. Clique em **Estado da instância > Reiniciar instância**.
3. Confirme a ação na janela de diálogo.
4. Aguarde o status mudar para **Em execução** — isso leva entre 2 e 5 minutos.

> **Nota:** Use sempre a opção "Reiniciar" pelo Console AWS para reinicializações de rotina. Evite desligar o servidor pelo Windows (Iniciar > Desligar), pois isso pode gerar estado inconsistente em algumas configurações de instância.

---

### Fase 3 — Verificação pós-reinicialização

**Passo 6 — Confirmar reinicialização bem-sucedida no Console**

1. Verifique que a instância retornou ao status **Em execução**.
2. Confirme que as verificações de status voltaram a **2/2 verificações aprovadas**.
3. Anote o novo **Endereço IPv4 público** (pode ter mudado se não houver Elastic IP).

**Passo 7 — Reconectar via RDP e verificar serviços**

1. Conecte-se novamente via RDP usando o endereço IP atualizado.
2. Aguarde o carregamento completo do ambiente (Server Manager pode levar 1–2 minutos adicionais).
3. Abra o **Gerenciador de Serviços** (`services.msc`) e confirme que todos os serviços críticos estão **Em execução**.
4. Verifique o **Visualizador de Eventos** (`eventvwr.msc`) em **Logs do Windows > Sistema** para identificar erros ocorridos durante o boot.

**Passo 8 — Validar funcionamento das aplicações**

1. Inicie manualmente qualquer serviço ou aplicação que não suba automaticamente.
2. Realize um teste funcional básico da aplicação principal hospedada na instância.
3. Confirme que os usuários conseguem acessar normalmente.

---

## 6. Registro de execução

Ao concluir o procedimento, registre as seguintes informações:

| Campo | Informação |
|---|---|
| Data e hora de início | |
| Data e hora de conclusão | |
| Motivo da reinicialização | |
| IP antes da reinicialização | |
| IP após a reinicialização | |
| Erros encontrados | |
| Ações corretivas tomadas | |
| Executado por | |

---

## 7. Situações de exceção

**Se a instância não voltar ao status "Em execução" em 10 minutos:**
1. Verifique o Console AWS em busca de mensagens de erro.
2. Tente a opção **Parar instância** e, após confirmação de parada, **Iniciar instância**.
3. Se o problema persistir, abra chamado no suporte AWS ou verifique o painel de status de serviços em [https://health.aws.amazon.com](https://health.aws.amazon.com).

**Se os serviços críticos não subirem automaticamente após o boot:**
1. Abra o `services.msc` e inicie o serviço manualmente.
2. Verifique se o tipo de inicialização está configurado como **Automático**.
3. Consulte o Visualizador de Eventos para identificar o motivo da falha.

**Se o RDP não conectar após reinicialização:**
1. Aguarde 5 minutos adicionais — instâncias Windows podem demorar para ter o RDP disponível.
2. Verifique se o IP público mudou e atualize o endereço na conexão.
3. Confirme as regras do Security Group no Console AWS.

---

## 8. Referências

- Documentação AWS EC2: [https://docs.aws.amazon.com/ec2](https://docs.aws.amazon.com/ec2)
- Tutorial interno: Como criar e configurar uma instância EC2 Windows Server
- FAQ interno: Suporte Técnico AWS EC2 com Windows Server

---

*Versão 1.0 — Elaborado com base em experiência prática de administração de infraestrutura AWS.*
