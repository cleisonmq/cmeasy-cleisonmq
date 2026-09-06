# Auditoria em Aplicativos Web (Websites)

A **Auditoria em Aplicativos Web** analisa um ou mais sites/serviços web e
produz um relatório com o que está exposto: portas e serviços, diretórios e
ficheiros acessíveis, tecnologias e WAF/CMS detetados, certificado TLS e —
quando existe — **base de dados exposta por SQL injection**.

> **Uso autorizado apenas.** Esta funcionalidade é intrusiva (faz pedidos
> reais e testes de segurança ao alvo). Use-a **exclusivamente** em sites
> seus ou para os quais tenha autorização explícita por escrito.

**Onde encontrar:** `Comandos` → `Auditoria do Sistema` →
`Auditoria em Aplicativos Web (Websites)`.

---

## Como indicar os alvos (sintaxe)

Ao iniciar a **Varredura de website**, o CMEasy pergunta como quer indicar
o(s) alvo(s):

- **[1] Apenas um host** — escreve um único IP ou domínio.
- **[2] Vários hosts** — depois escolhe:
  - **Listar manualmente** — vários alvos **separados por vírgula**.
  - **Utilizar um arquivo** — indica o caminho de um ficheiro de texto
    (com autocompletar por `Tab`).

### Formato de cada alvo

Cada alvo pode ser um domínio ou um IP, com ou sem porta:

| Formato | Exemplo | Significado |
|---|---|---|
| Domínio | `empresa.pt` | Descoberta automática de portas |
| Subdomínio | `api.empresa.pt` | Descoberta automática de portas |
| IPv4 | `10.0.1.2` | Descoberta automática de portas |
| Domínio com porta | `empresa.pt:8080` | Audita **só** a porta 8080 |
| IPv4 com porta | `10.0.1.2:8080` | Audita **só** a porta 8080 |
| IPv6 com porta | `[::1]:443` | IPv6 tem de vir entre `[ ]` para indicar porta |

Notas de sintaxe:
- Se **não** indicar porta, o CMEasy faz **descoberta automática** das portas
  web do alvo.
- Um IPv6 **sem** porta escreve-se tal como é (ex.: `2001:db8::1`); para lhe
  indicar uma porta use a notação com colchetes (`[2001:db8::1]:443`).
- Vários alvos manuais separam-se por vírgula:
  `empresa.pt, api.empresa.pt, 10.0.1.2`

### Formato do ficheiro de hosts

Quando escolhe **Utilizar um arquivo**, o ficheiro deve ter:

- **Um host por linha** (cada linha no formato de alvo acima).
- Linhas em branco são ignoradas.
- Linhas iniciadas por `#` são tratadas como **comentários** e ignoradas.
- Espaços no início/fim de cada linha são removidos.

**Exemplo de ficheiro `hosts.txt`:**

```text
# Sites de produção
empresa.pt
api.empresa.pt:8080

# Servidores internos
10.0.1.2
[2001:db8::10]:443
```

---

## O que a auditoria verifica

Para cada alvo, o CMEasy executa (de forma automática e sequencial):

1. **Resolução e conectividade** — resolve o DNS e confirma se o host
   responde. Um alvo que não resolva/responda é assinalado e ignorado (não é
   testado às cegas).
2. **Descoberta de portas e serviços** — identifica portas web abertas e os
   serviços/versões associados.
3. **Diretórios e ficheiros expostos** — procura páginas, diretórios e
   ficheiros acessíveis publicamente (ex.: painéis de administração, backups,
   `.git`, ficheiros de configuração), incluindo os que devolvem `200 OK`.
4. **Tecnologias, WAF e CMS** — deteta a stack do site (servidor, framework,
   CMS como WordPress) e se existe uma firewall aplicacional (WAF) à frente
   (Cloudflare, Akamai, AWS/Azure WAF, Imperva…).
5. **Certificado TLS** — emissor, validade e avisos (autoassinado, expirado,
   nome que não corresponde).
6. **Base de dados exposta (SQL Injection)** — ver secção seguinte.

---

## Base de Dados Exposta (SQL Injection)

O CMEasy testa se o site tem uma **base de dados exposta por SQL injection**
e, se tiver, recolhe os **metadados** dessa base de dados.

**O que recolhe (apenas leitura de metadados):**

- Se o site é **injetável** (e qual o parâmetro vulnerável);
- O **SGBD** (ex.: MySQL, PostgreSQL, MSSQL…);
- A **versão** do SGBD (banner);
- A **base de dados atual** da aplicação;
- A **lista de bases de dados** acessíveis.

**O que NÃO faz:** não extrai o conteúdo das tabelas (linhas de dados) nem
executa comandos no servidor. A auditoria **demonstra** a exposição — mostra
o nome e a versão da base de dados — mas **não exfiltra nem altera** registos.

### Exemplo no relatório

Quando há exposição:

```text
Base de Dados Exposta (SQL Injection)
Estado: BASE DE DADOS EXPOSTA — SQL injection confirmada.
Parâmetro vulnerável: id
SGBD: MySQL >= 5.6
Versão (banner): 5.7.31-0ubuntu0.18.04.1
Base de dados atual: shopdb
Bases de dados acessíveis (3):
  - information_schema
  - shopdb
  - mysql

AVISO: esta exposição permite ler/alterar dados. Corrigir com queries
parametrizadas (prepared statements) e privilégios mínimos na conta da
aplicação.
```

Quando **não** há exposição:

```text
Base de Dados Exposta (SQL Injection)
Estado: sem base de dados exposta — não injetável (nenhum parâmetro vulnerável).
```

Se a ferramenta de teste não estiver instalada, o estado indica-o
(`não testado — sqlmap não instalado`) e o resto da auditoria continua
normalmente.

---

## O relatório

- A auditoria gera um relatório em texto na pasta de relatórios do CMEasy do
  utilizador (`.../Documentos/cmeasy/relatorios/`), com o nome
  `relatorio_de_varredura_de_sites.txt`. A cada nova execução é criado um
  novo ficheiro (o histórico anterior nunca é apagado).
- Pode consultá-lo mais tarde pela opção **Ver relatório** do mesmo menu.
- Cada host tem a sua própria secção, com todos os pontos acima
  (portas, diretórios, tecnologias, TLS e base de dados).

---

## Boas práticas de correção

Se a auditoria encontrar problemas:

- **Base de dados exposta:** use *queries* parametrizadas (prepared
  statements) e dê à conta da aplicação apenas os privilégios mínimos
  necessários.
- **Diretórios/ficheiros expostos:** remova backups, `.git` e ficheiros de
  configuração da raiz web; restrinja o acesso a painéis de administração.
- **Sem WAF / TLS fraco:** considere uma firewall aplicacional e um
  certificado válido (não autoassinado) em produção.
