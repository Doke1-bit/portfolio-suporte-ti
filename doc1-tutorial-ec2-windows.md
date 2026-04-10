# Tutorial — Como criar e configurar uma instância EC2 com Windows Server na AWS

**Tipo:** Tutorial técnico passo a passo  
**Nível:** Iniciante / Intermediário  
**Tempo estimado:** 30–45 minutos  
**Pré-requisitos:** Conta ativa na AWS, acesso à internet, cliente RDP instalado

---

## Objetivo

Este tutorial descreve como provisionar uma instância EC2 com Windows Server na AWS, configurar as regras de rede (Security Group) e acessar o servidor remotamente via RDP (Remote Desktop Protocol).

---

## Etapa 1 — Acessar o console da AWS e navegar até o EC2

1. Acesse [https://console.aws.amazon.com](https://console.aws.amazon.com) e faça login com sua conta.
2. No campo de busca do topo, digite **EC2** e clique no serviço.
3. No painel lateral esquerdo, clique em **Instâncias** e depois em **Executar instâncias** (botão laranja no canto superior direito).

---

## Etapa 2 — Escolher o sistema operacional (AMI)

1. No campo **Nome**, dê um nome descritivo para sua instância (ex: `servidor-windows-teste`).
2. Em **Imagem de aplicação e sistema operacional (AMI)**, clique em **Windows**.
3. Selecione **Windows Server 2022 Base** (ou a versão mais recente disponível).

> **Atenção:** Instâncias Windows consomem mais RAM e disco que instâncias Linux. Para uso no Free Tier, o tipo `t2.micro` é elegível, mas tem recursos limitados.

---

## Etapa 3 — Selecionar o tipo de instância

1. Em **Tipo de instância**, selecione **t2.micro** para uso dentro do Free Tier (elegível por 12 meses em contas novas).
2. Verifique se a coluna **Free Tier elegível** está marcada antes de prosseguir.

| Tipo | vCPU | RAM | Uso recomendado |
|---|---|---|---|
| t2.micro | 1 | 1 GB | Testes, servidores leves |
| t2.medium | 2 | 4 GB | Aplicações com carga moderada |
| t2.large | 2 | 8 GB | Servidores com múltiplos usuários |

---

## Etapa 4 — Criar ou selecionar um par de chaves

1. Em **Par de chaves (login)**, clique em **Criar novo par de chaves**.
2. Dê um nome (ex: `chave-windows-teste`), mantenha o formato **RSA** e extensão **.pem**.
3. Clique em **Criar par de chaves** — o arquivo `.pem` será baixado automaticamente.

> **Importante:** Guarde o arquivo `.pem` em local seguro. Ele é necessário para recuperar a senha de administrador do Windows Server. Não é possível recuperá-lo após o download.

---

## Etapa 5 — Configurar o Security Group (regras de rede)

1. Em **Configurações de rede**, clique em **Editar**.
2. Mantenha a regra padrão de **RDP (porta 3389)** para acesso remoto.
3. Em **Tipo de origem**, selecione **Meu IP** — isso restringe o acesso somente ao seu endereço IP atual, aumentando a segurança.

> **Boas práticas de segurança:**
> - Nunca libere a porta 3389 para `0.0.0.0/0` (qualquer origem) em ambientes de produção.
> - Remova regras desnecessárias após o uso.
> - Considere usar **AWS Systems Manager Session Manager** como alternativa mais segura ao RDP em ambientes corporativos.

---

## Etapa 6 — Configurar armazenamento

1. Em **Configurar armazenamento**, o padrão é **30 GB gp3** — suficiente para testes.
2. Mantenha as configurações padrão para uso no Free Tier.

---

## Etapa 7 — Executar a instância

1. Revise o **Resumo** no painel direito.
2. Clique em **Executar instância**.
3. Aguarde a inicialização — o status mudará de `Pendente` para `Em execução` em 2 a 5 minutos.

---

## Etapa 8 — Recuperar a senha de administrador

1. Na lista de instâncias, clique com o botão direito sobre sua instância e selecione **Segurança → Obter senha do Windows**.
2. Clique em **Carregar arquivo de chave privada** e selecione o arquivo `.pem` baixado na Etapa 4.
3. Clique em **Descriptografar senha**.
4. Anote ou copie a senha exibida — ela será usada no login RDP.

> **Observação:** A senha pode levar até 4 minutos para ficar disponível após o primeiro boot da instância.

---

## Etapa 9 — Conectar via RDP

1. Na lista de instâncias, selecione sua instância e clique em **Conectar**.
2. Acesse a aba **Cliente RDP** e clique em **Baixar arquivo de área de trabalho remota** — isso gera um arquivo `.rdp` pré-configurado.
3. Abra o arquivo `.rdp`. Quando solicitado, insira:
   - **Usuário:** `Administrator`
   - **Senha:** a senha recuperada na Etapa 8.
4. Aceite o certificado de segurança e aguarde a conexão.

---

## Verificação final

Após conectar, confirme que o ambiente está funcionando:

- [ ] Área de trabalho do Windows Server carregou corretamente
- [ ] Gerenciador de Tarefas abre sem erros (`Ctrl + Shift + Esc`)
- [ ] Acesso à internet funcionando (abrir navegador e testar)
- [ ] Painel de controle do servidor (Server Manager) inicializou

---

## Solução de problemas comuns

| Sintoma | Causa provável | Ação recomendada |
|---|---|---|
| RDP não conecta | Porta 3389 bloqueada no Security Group | Verificar regras de entrada no Security Group |
| "Credenciais inválidas" | Senha incorreta ou expirada | Repetir Etapa 8 para recuperar a senha |
| Instância não inicializa | Limite de Free Tier atingido | Verificar uso em Billing > Free Tier |
| Conexão lenta ou travando | Instância t2.micro sobrecarregada | Verificar uso de CPU/RAM pelo CloudWatch |
| Arquivo .pem perdido | Chave não foi salva | Não é recuperável — criar nova instância |

---

## Próximos passos sugeridos

- Instalar aplicações no servidor (ex: servidor de jogos, servidor web IIS)
- Configurar backups automáticos via snapshots EBS
- Criar um Elastic IP para manter o endereço fixo entre reinicializações
- Monitorar métricas de uso pelo Amazon CloudWatch

---

*Documentação elaborada com base em experiência prática de administração de instâncias EC2 Windows na AWS.*
