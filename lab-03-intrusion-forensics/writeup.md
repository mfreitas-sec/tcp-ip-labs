# Lab 03 — Forense de Intrusão

**Arquivo analisado:** `invasao.pcap`
**Ferramentas:** Wireshark, tcpdump, `file`, `xdg-open`, OUI Lookup
**Técnicas:** Intrusão FTP, file carving via RAW stream, reconstrução de arquivo por magic bytes

> Este foi o laboratório mais desafiador do módulo. A resolução exigiu pensar além das ferramentas e entender **o comportamento do protocolo e da rede** para encontrar a evidência crítica.

---

## 🎯 Objetivos

- Identificar MAC e fabricante do atacante
- Identificar IP do alvo
- Descobrir portas abertas e versão do serviço comprometido
- Recuperar as credenciais utilizadas
- Identificar o arquivo roubado
- **Recuperar o conteúdo do arquivo extraído da rede**

---

## 🔍 Metodologia e Resolução

### 1. Identificação do atacante

Ao carregar o `.pcap` no Wireshark, os pacotes iniciais revelaram os endereços de camada 2 e 3 com clareza através da coluna **Source**:

- **MAC do atacante:** `XX:XX:XX:XX:XX:XX` *(omitido)*
- **Fabricante identificado via:** [Wireshark OUI Lookup](https://www.wireshark.org/tools/oui-lookup.html) — os primeiros 3 octetos do MAC foram consultados na base OUI, confirmando o fabricante

---

### 2. Identificação do IP do alvo via ICMP

A análise dos pacotes de protocolo **ICMP** (ping/echo) revelou de forma clara os endereços de origem e destino, permitindo identificar o IP do alvo sem ambiguidade.

> ICMP é frequentemente usado em reconhecimento inicial (host discovery) — sua presença em um pcap de intrusão indica que o atacante realizou varredura de hosts ativos antes de avançar.

---

### 3. Enumeração de portas abertas

Utilizando `tcpdump` com filtro de flags FIN (mesma lógica do Lab 01), as portas abertas foram identificadas:

```bash
tcpdump -vnr invasao.pcap src host [ATTACKER_IP] | cut -d "," -f1 | grep -v "tos" | grep "F\."
```

**Portas abertas identificadas:** `80, 443, 21011, 51125`

---

### 4. Identificação da versão do serviço comprometido

Cada porta foi analisada individualmente via **Follow TCP Stream** no Wireshark. Na porta `21011`, o banner do servidor foi exposto na inicialização da conexão, revelando a versão exata do serviço.

```
Serviço: [REDACTED] — versão identificada no banner inicial da conexão
```

---

### 5. Recuperação das credenciais — análise de RST e reconexão

Esta etapa foi a mais trabalhosa. A porta `21011` continha diversas tentativas de login, porém a conexão era interrompida com flag **RST** antes da autenticação ser completada em várias sessões.

**Comportamento observado:**
```
[Tentativa 1] → ... → RST (conexão encerrada pelo servidor)
[Tentativa 2] → ... → RST
...
[Tentativa N] → LOGIN ACEITO ✓
```

A solução foi descer até o **final das requisições** na porta `21011` e aplicar **Follow TCP Stream** na última sessão — onde a autenticação foi bem-sucedida e as credenciais estavam visíveis em texto claro.

```
Credenciais: [REDACTED]
```

---

### 6. Identificação do arquivo roubado

Na mesma stream onde as credenciais foram encontradas, a sequência de comandos FTP revelou a solicitação e transferência de um arquivo específico:

```ftp
RETR manual.txt
```

**Arquivo roubado:** `manual.txt`

---

### 7. Recuperação do conteúdo — o desafio final ⚡

Esta foi a parte mais desafiadora do laboratório.

**Tentativa inicial (incorreta):**

O caminho natural seria acessar o serviço FTP diretamente no IP do alvo pela porta `21011`. Porém, ao tentar conectar, a porta não respondia. Um ping com destino a esse host retornava `DIFFERENT ADDRESS` — indicando que a porta `21011` **não era um serviço permanente**, apenas um canal temporário aberto durante a intrusão.

> O SysAdmin havia trocado as credenciais FTP após a invasão — mas isso era uma distração. O ponto real era outro.

**Análise crítica das dicas do lab:**

O enunciado trouxe perguntas fundamentais:
- *A transferência aconteceu durante a captura do tráfego?* → **Sim**
- *O tráfego estava criptografado?* → **Não**
- *Se o FTP não é opção, onde o arquivo pode estar?*

**A porta ignorada: `51125`**

A porta `51125` havia sido identificada como aberta, mas descartada por aparentar não ter relevância. Ao aplicar **Follow TCP Stream** nela com encoding **ASCII**, a saída retornava caracteres corrompidos e ilegíveis — o que normalmente levaria ao abandono dessa pista.

**A sacada:**

Caracteres ilegíveis em ASCII indicam dados **binários** — não texto. Isso sugere que o conteúdo era um arquivo binário sendo transferido. A solução foi:

**1.** Abrir o Follow TCP Stream da porta `51125`

**2.** Alterar o encoding de `ASCII` para **`RAW`**

**3.** Salvar o conteúdo raw como arquivo:
```bash
# Arquivo salvo como manual.pdf
```

**4.** Validar o tipo do arquivo pelos magic bytes:
```bash
file manual.pdf
# Saída: manual.pdf: PDF document, version X.X
```

**5.** Abrir o arquivo reconstruído:
```bash
xdg-open manual.pdf
```

O PDF abriu corretamente, revelando a chave:

```
KEY: [FLAG_REDACTED]
```

---

## 📌 Conclusões e aprendizados

| Ponto | Aprendizado |
|-------|------------|
| RST em FTP | Múltiplas tentativas com RST indicam força bruta ou instabilidade — analisar a última sessão |
| Porta "irrelevante" | Nenhuma porta aberta deve ser ignorada em forense — pode conter a evidência principal |
| ASCII vs RAW no Stream | Dados binários aparecem corrompidos em ASCII — trocar para RAW é essencial para file carving |
| `file` command | Identifica tipo real pelo cabeçalho (magic bytes) — independente da extensão do arquivo |
| Serviços temporários | Portas abertas durante intrusão podem não estar mais ativas — o arquivo já foi capturado na rede |
| Pensamento investigativo | A solução não estava na ferramenta, estava em **entender por que o tráfego estava quebrado** |

### Lição principal

> Em forense de rede, caracteres ilegíveis não significam "sem dados úteis" — significam **dados em formato não-texto**. A pergunta correta não é *"o que está aqui?"*, mas *"em que formato está aqui?"*

---

## 🔗 Navegação

[← Lab 02](../lab-02-byte-analysis/writeup.md) | [↑ Voltar ao início](../README.md)
