# Lab 02 — Análise de Sequência de Bytes

**Arquivo analisado:** `bytes.txt`
**Ferramentas:** `printf`, `xxd`, Firefox
**Técnicas:** Hex-to-decimal conversion, HTTP header forensics, credential extraction, web authentication

---

## 🎯 Objetivos

A partir de uma sequência de bytes bruta, extrair:

- Socket de origem e destino
- Flags TCP ativas
- Porta de origem
- Referer da requisição HTTP
- User-Agent (navegador utilizado)
- Credenciais de acesso (usuário e senha)
- Chave de acesso pós-autenticação

---

## 🔍 Metodologia e Resolução

### 1. Conversão de bytes — identificação do socket

O arquivo `bytes.txt` continha uma sequência hexadecimal bruta representando um pacote de rede. A primeira etapa foi converter manualmente os campos de cabeçalho para decimal, identificando endereços e portas.

Ferramenta utilizada: `printf` com formatação hexadecimal

```bash
printf "%d\n" 0x[BYTES_DO_CAMPO]
```

Este processo foi aplicado nos campos correspondentes ao **IP de origem**, **IP de destino** e **portas**, conforme a estrutura do cabeçalho TCP/IP.

**Resultados obtidos:**
- Socket de origem: `[REDACTED]`
- Socket de destino: `[REDACTED]`
- Flags ativas: identificadas pela análise do byte de controle TCP
- Porta de origem: extraída do campo correspondente no cabeçalho

---

### 2. Decodificação do payload — recuperação do conteúdo HTTP

Para visualizar o conteúdo do payload (camada de aplicação), o arquivo foi convertido de hexadecimal para texto legível usando `xxd`:

```bash
xxd -r -p bytes.txt
```

A saída revelou uma **requisição HTTP completa**, contendo:

- **Referer:** URL de origem da requisição *(omitida)*
- **User-Agent:** identificação do navegador utilizado *(omitido)*
- **Credenciais:** transmitidas em texto claro no corpo da requisição

> **Observação técnica:** A presença de credenciais em texto claro em uma requisição HTTP (sem HTTPS) é uma vulnerabilidade crítica. Qualquer captura de tráfego na rede intermediária expõe os dados de autenticação completamente.

---

### 3. Acesso e obtenção da chave

Com as credenciais extraídas e o endereço do serviço identificado nos bytes, o acesso foi realizado via browser:

1. Navegação até o socket identificado
2. Autenticação com as credenciais recuperadas
3. Obtenção da chave de acesso

```
KEY: [FLAG_REDACTED]
```

---

## 📌 Conclusões e aprendizados

| Técnica | Descrição |
|---------|-----------|
| `printf "%d\n" 0x...` | Converte campos hexadecimais do cabeçalho para decimal (IPs, portas, flags) |
| `xxd -r -p` | Reconstrói conteúdo legível a partir de dump hexadecimal |
| HTTP sem TLS | Credenciais e headers trafegam expostos — capturáveis por qualquer host na rede |
| Análise manual de cabeçalho | Entender a estrutura TCP/IP permite extrair informações sem ferramentas gráficas |

### Por que isso importa em Pentest?

A capacidade de analisar bytes brutos é fundamental em cenários onde ferramentas gráficas não estão disponíveis — como em shells reversas, ambientes mínimos ou análise de arquivos offline. Entender a estrutura do cabeçalho TCP/IP na camada de bytes é o que diferencia um analista que usa ferramentas de um que **entende o protocolo**.

---

## 🔗 Navegação

[← Lab 01](../lab-01-internal-scan/writeup.md) | [Lab 03 →](../lab-03-intrusion-forensics/writeup.md)
