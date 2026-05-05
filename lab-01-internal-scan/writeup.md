# Lab 01 — Análise de Scan Interno

**Arquivo analisado:** `scan_interno.pcap`
**Ferramentas:** Wireshark, tcpdump, grep, cut
**Técnicas:** MAC/IP identification, TCP flag analysis, FTP stream inspection

---

## 🎯 Objetivos

- Identificar o endereço físico (MAC) e lógico (IP) do atacante
- Identificar o endereço lógico do alvo
- Descobrir quais portas estavam abertas no alvo
- Identificar o serviço comprometido e sua versão
- Recuperar as credenciais utilizadas no ataque

---

## 🔍 Metodologia e Resolução

### 1. Identificação do atacante e alvo

Ao carregar o arquivo no Wireshark, a análise inicial do tráfego revelou comunicação entre **dois hosts distintos**. Observando o campo **Source** nos pacotes, foi possível identificar rapidamente:

- **MAC do atacante:** `XX:XX:XX:XX:XX:XX` *(omitido — dado sensível do lab)*
- **IP do atacante:** `192.168.0.X` *(omitido)*
- **IP do alvo:** `192.168.0.X` *(omitido)*

A identificação do fabricante da interface de rede foi feita diretamente pelo Wireshark, que resolve o prefixo OUI do endereço MAC automaticamente — revelando tratar-se de um adaptador **IBM**.

---

### 2. Identificação de portas abertas — uso de flags TCP

Esta etapa exigiu raciocínio mais aprofundado. A abordagem inicial com tcpdump retornou saída ruidosa e de difícil leitura:

```bash
# Tentativa inicial — saída pouco legível
tcpdump -vnr scan_interno.pcap host [ATTACKER_IP] | grep "F\."
```

Após refinamento, o filtro correto isolou apenas pacotes com a **flag FIN** originados do atacante, com saída limpa via `cut`:

```bash
# Filtro otimizado
tcpdump -vnr scan_interno.pcap src host [ATTACKER_IP] | cut -d "," -f1 | grep -v "tos" | grep "F\."
```

**Raciocínio por trás do filtro:**

> A flag **FIN** indica encerramento de conexão TCP. Se houve encerramento, houve conexão estabelecida. Se houve conexão estabelecida, a porta estava **aberta** no alvo.

Portas identificadas como abertas: `80, 2119, 2120, 2222, 3306, 8089`

---

### 3. Identificação do serviço e versão

Com as portas em mãos, cada uma foi inspecionada no Wireshark através do recurso **Follow TCP Stream**, que reconstrói o conteúdo da sessão TCP em texto legível.

Em uma das portas, o banner do serviço foi exposto diretamente na conexão, revelando:

```
Service: ProFTPD 1.3.5rc3
```

> **Nota técnica:** ProFTPD 1.3.5rc3 é uma versão com vulnerabilidades conhecidas e catalogadas no CVE. Em um teste de penetração real, esta versão seria imediatamente marcada para exploração.

---

### 4. Recuperação de credenciais

Analisando o TCP Stream da porta do serviço FTP identificado, as credenciais de autenticação foram transmitidas em **texto claro** — comportamento esperado do protocolo FTP padrão (sem TLS).

```
Credenciais: [REDACTED] (omitidas por questões éticas)
```

---

## 📌 Conclusões e aprendizados

| Ponto | Aprendizado |
|-------|------------|
| Flags TCP | FIN indica conexão encerrada → porta estava aberta |
| Filtragem tcpdump | Combinar `src host` + `cut` + `grep` reduz ruído drasticamente |
| FTP sem TLS | Credenciais trafegam em texto claro — visíveis em qualquer captura |
| Banner grabbing | Serviços expõem versão no handshake inicial — base do fingerprinting |
| Follow TCP Stream | Recurso essencial para reconstruir sessões e visualizar conteúdo |

---

## 🔗 Próximo lab

[Lab 02 → Análise de Sequência de Bytes](../lab-02-byte-analysis/writeup.md)
