# 🔬 TCP/IP Labs — Network Forensics & Traffic Analysis

> Repositório de write-ups dos laboratórios práticos do módulo **TCP/IP para Pentesters** da [DESEC Security](https://desecsecurity.com/).

---

## 📋 Sobre este repositório

Este repositório documenta a resolução de laboratórios práticos de análise de tráfego de rede, forense de pacotes e investigação de intrusões. Cada lab utiliza arquivos `.pcap` reais e sequências de bytes capturadas em cenários simulados de ataque.

O objetivo é demonstrar capacidade de análise de tráfego, identificação de comportamento malicioso e extração de evidências — habilidades fundamentais em **Blue Team**, **SOC**, **Pentest** e **DFIR**.

---

## 🛠️ Ferramentas utilizadas

| Ferramenta | Uso |
|-----------|-----|
| `Wireshark` | Análise visual de pacotes, TCP Stream, filtros de protocolo |
| `tcpdump` | Filtragem avançada via linha de comando |
| `xxd` | Conversão e leitura de dumps hexadecimais |
| `printf` | Conversão manual de valores hexadecimais para decimal |
| `cut / grep` | Filtragem e tratamento de saída no terminal |
| `file` | Identificação de tipo de arquivo por magic bytes |
| `xdg-open` | Abertura de arquivos reconstruídos |
| OUI Lookup (Wireshark) | Identificação de fabricante por endereço MAC |

---

## 📁 Laboratórios

| # | Lab | Técnicas | Dificuldade |
|---|-----|----------|-------------|
| 01 | [Análise de Scan Interno](./lab-01-internal-scan/writeup.md) | PCAP analysis, tcpdump, TCP flags, TCP Stream | ⭐⭐ |
| 02 | [Análise de Sequência de Bytes](./lab-02-byte-analysis/writeup.md) | Hex to decimal, xxd, HTTP forensics, credenciais | ⭐⭐⭐ |
| 03 | [Forense de Intrusão](./lab-03-intrusion-forensics/writeup.md) | PCAP completo, FTP analysis, file carving, RAW stream | ⭐⭐⭐⭐ |

---

## 🧠 Habilidades demonstradas

- Identificação de **endereços físicos (MAC) e lógicos (IP)** de atacantes e alvos em capturas de tráfego
- Uso de **flags TCP** (SYN, FIN, RST, ACK) para inferir estado de portas e sessões
- **Reconstrução de sessões TCP** via Follow TCP Stream no Wireshark e tcpdump
- Análise de **credenciais em protocolos não criptografados** (FTP, HTTP)
- **Conversão e interpretação de sequências de bytes** (hex → decimal → ASCII)
- **File carving** — reconstrução de arquivos a partir de streams de rede brutos
- Identificação de **fabricantes de hardware** via OUI lookup
- Raciocínio investigativo em cenários de **intrusão real simulada**

---

## ⚠️ Aviso

Os arquivos `.pcap` utilizados nestes laboratórios são propriedade da DESEC Security e disponibilizados exclusivamente para alunos matriculados. IPs, MACs e credenciais reais foram omitidos dos write-ups por questões éticas e de segurança.

---

## 👤 Autor

**Murilo Freitas** — [@mfreitas-sec](https://github.com/mfreitas-sec)
Em transição para Pentest & Red Team | Estudando para eJPT
