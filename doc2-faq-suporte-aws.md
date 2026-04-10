# FAQ — Suporte Técnico: AWS EC2 com Windows Server

**Tipo:** Base de conhecimento / FAQ  
**Público-alvo:** Usuários e administradores iniciantes em AWS  
**Última atualização:** Abril 2026

---

## Categoria 1 — Acesso e conectividade

---

**Q: Não consigo conectar via RDP. O que verificar primeiro?**

Siga esta sequência de diagnóstico antes de escalar o chamado:

1. **Verifique o status da instância** no Console AWS — ela precisa estar com status `Em execução` e passar nas verificações de sistema (2/2 checks).
2. **Confirme o Security Group** — vá em Segurança > Security Groups e verifique se existe uma regra de entrada liberando a porta **TCP 3389** para o seu IP.
3. **Verifique o endereço IP** — instâncias EC2 recebem um IP público novo a cada reinicialização, a menos que um Elastic IP esteja associado. Confirme o IP atual na página da instância.
4. **Teste a porta** — use um verificador de porta online (ex: portchecker.co) para confirmar se a porta 3389 está acessível externamente.
5. **Aguarde o boot completo** — instâncias Windows levam entre 3 e 5 minutos para estar totalmente prontas após o início.

Se todos os pontos acima estiverem corretos e o problema persistir, verifique se o firewall interno do Windows Server não está bloqueando conexões RDP.

---

**Q: Meu IP mudou e perdi acesso ao servidor. Como resolver?**

Instâncias EC2 recebem um endereço IP público dinâmico, que muda toda vez que a instância é parada e reiniciada.

**Solução imediata:**
1. Acesse o Console AWS e localize sua instância.
2. Copie o novo **Endereço IPv4 público** exibido na página de detalhes.
3. Atualize o arquivo `.rdp` com o novo endereço ou conecte diretamente pelo Console em **Conectar > Cliente RDP**.

**Solução definitiva:**
Associe um **Elastic IP** à instância. O Elastic IP é um endereço fixo que permanece mesmo após reinicializações:
1. No painel EC2, vá em **Rede e Segurança > IPs Elásticos**.
2. Clique em **Alocar endereço IP elástico**.
3. Após alocar, clique em **Associar endereço** e selecione sua instância.

> **Atenção sobre custos:** O Elastic IP é gratuito enquanto estiver associado a uma instância em execução. Se a instância estiver parada ou o IP não estiver associado, haverá cobrança por hora.

---

**Q: A senha de Administrador não funciona no RDP. O que fazer?**

Verifique as situações abaixo:

- **Arquivo .pem incorreto:** A descriptografia da senha exige o arquivo `.pem` original usado na criação da instância. Arquivos diferentes não funcionam.
- **Instância recém-criada:** A senha pode levar até 4 minutos para ficar disponível após o primeiro boot. Aguarde e tente novamente.
- **Senha expirada por política do Windows:** O Windows Server pode forçar troca de senha após o primeiro login. Conecte via Console AWS (usando **Sessão do EC2 Connect** se disponível) para redefinir.
- **CapsLock ativo:** Verifique se o CapsLock não está alterando a digitação da senha.

---

## Categoria 2 — Desempenho e estabilidade

---

**Q: O servidor está lento e com alto uso de CPU. O que investigar?**

1. **Acesse o Gerenciador de Tarefas** (`Ctrl + Shift + Esc`) e identifique qual processo está consumindo CPU.
2. **Verifique o CloudWatch** no Console AWS — o gráfico de CPU da instância mostra se o pico é recorrente ou pontual.
3. **Instâncias t2.micro** têm créditos de CPU — uso sustentado acima de 10% por longos períodos pode causar throttling (limitação automática). Isso é comum em servidores com carga constante.

**Ações recomendadas:**
- Reiniciar o processo ou serviço que consome mais recursos.
- Avaliar upgrade para uma instância maior (ex: t2.medium) se o uso for legítimo e recorrente.
- Verificar se há processos desnecessários em execução na inicialização (`msconfig` > Inicialização).

---

**Q: O servidor travou e não responde mais ao RDP. Como recuperar?**

1. **Tente reiniciar via Console AWS:** Selecione a instância > **Estado da instância > Reiniciar**. Isso é equivalente a um reboot de SO e preserva os dados.
2. **Se o reboot não resolver:** Use **Estado da instância > Parar** e em seguida **Iniciar**. Isso reinicia o hardware subjacente e resolve a maioria dos travamentos.
3. **Verifique os logs de sistema** após a recuperação: no Gerenciador de Eventos do Windows (`eventvwr.msc`), filtre por erros críticos no período do travamento.

> **Diferença entre Reiniciar e Parar/Iniciar:** "Reiniciar" mantém o mesmo servidor físico e IP. "Parar/Iniciar" pode mover a instância para outro servidor físico e sempre troca o IP público (se não houver Elastic IP).

---

**Q: O servidor consome toda a RAM disponível. Como liberar memória?**

1. Identifique o processo que consome mais memória pelo Gerenciador de Tarefas (aba Processos, ordenar por Memória).
2. Verifique se há **memory leaks** — processos que crescem continuamente ao longo do tempo sem liberar memória. Reiniciar o processo geralmente resolve temporariamente.
3. Configure uma **tarefa agendada** no Task Scheduler para reiniciar o processo problemático em horário de baixo uso (ex: 4h da manhã).
4. Avalie reduzir a quantidade de serviços em execução simultânea na instância.

---

## Categoria 3 — Segurança

---

**Q: Como sei se alguém tentou acessar meu servidor sem autorização?**

Verifique os **logs de segurança do Windows:**

1. Abra o **Visualizador de Eventos** (`eventvwr.msc`).
2. Navegue até **Logs do Windows > Segurança**.
3. Filtre pelo ID de evento **4625** — cada ocorrência representa uma tentativa de login com falha.
4. Verifique o campo **Endereço de rede de origem** — IPs desconhecidos com muitas tentativas indicam ataque de força bruta.

**Ações preventivas recomendadas:**
- Restrinja a porta 3389 no Security Group para apenas o seu IP.
- Considere alterar a porta RDP padrão para dificultar varreduras automatizadas.
- Ative o bloqueio de conta após tentativas consecutivas falhas (Política de Segurança Local > Política de Bloqueio de Conta).

---

**Q: Preciso liberar uma porta adicional no servidor. Como fazer?**

1. No Console AWS, acesse **EC2 > Security Groups**.
2. Selecione o Security Group associado à sua instância.
3. Clique em **Editar regras de entrada > Adicionar regra**.
4. Preencha: **Tipo** (Custom TCP), **Intervalo de portas** (número da porta), **Origem** (seu IP ou faixa necessária).
5. Clique em **Salvar regras**.

> **Importante:** Liberar uma porta no Security Group é necessário, mas não suficiente. O firewall interno do Windows Server (Windows Defender Firewall) também precisa ter uma regra de entrada correspondente. Acesse **Painel de Controle > Firewall do Windows Defender > Configurações Avançadas > Regras de Entrada > Nova Regra**.

---

## Categoria 4 — Custos e Free Tier

---

**Q: Como evitar cobranças inesperadas na AWS?**

- **Pare instâncias quando não usar:** Instâncias paradas não geram cobrança de computação, apenas de armazenamento EBS (que é mínimo no Free Tier).
- **Monitore o uso do Free Tier:** Acesse **Billing > Free Tier** para ver quanto do limite mensal você já consumiu.
- **Configure alertas de billing:** Em **Billing > Preferências de faturamento**, ative alertas e crie um alarme no CloudWatch para ser notificado quando os custos ultrapassarem um valor definido (ex: $1).
- **Elastic IP parado:** Lembre-se que Elastic IPs não associados geram cobrança. Libere IPs que não estiver usando.

---

*Este FAQ foi elaborado com base em situações reais de administração e suporte a ambientes AWS EC2 com Windows Server.*
