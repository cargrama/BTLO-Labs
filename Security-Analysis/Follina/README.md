# Estudo de Caso: Analisando a Vulnerabilidade Follina (CVE-2022-30190)

## 1. Introdução: O que é a Follina?
A vulnerabilidade **Follina (CVE-2022-30190)** é uma falha crítica de Execução Remota de Código (RCE) que foi descoberta em 2022. O que a torna fascinante e perigosa é que ela não depende de macros maliciosas para funcionar. 

Ela explora o **MSDT (Microsoft Support Diagnostic Tool)**, uma ferramenta legítima do Windows, através de um link de "template remoto" dentro de documentos do Office. Quando a vítima abre o arquivo, o Windows invoca o MSDT, que acaba executando comandos maliciosos enviados pelo atacante.

---

## 2. Metodologia e Preparação do Ambiente
Como analista iniciante, minha prioridade foi a **segurança** e a **higiene forense** para evitar que o malware afetasse meu computador real.

### 2.1. Lab Setup
* **VM:** Windows 10 isolada.
* **Snapshots:** Criei um ponto de restauração antes de qualquer análise para garantir que a VM pudesse ser resetada.
* **Rede:** Iniciei com NAT (para baixar ferramentas) e mudei para **Host-Only** logo após receber o arquivo malicioso, isolando totalmente o vírus.

### 2.2. Transferência Segura (Postura de Blue Team)
Embora a VM tivesse internet, baixei o arquivo no host e criei um servidor web temporário com Python:
`python3 -m http.server 80`
**Justificativa:** Isso evita o uso de navegadores na VM, que podem ser vetores de outros ataques, e mantém o ambiente de análise mais "puro".

> **📍 [INCLUIR PRINT 1: Terminal com o comando Python no host e o download sendo feito na VM]**

---

## 3. Passo a Passo da Investigação (Resolução BTLO)

### Fase 1: Identificação do Inimigo (Questões 1 e 2)
A primeira tarefa foi garantir que o arquivo não estava corrompido e descobrir sua identidade real.
* **Ação:** Usei o PowerShell para gerar o hash.
* **Comando:** `Get-FileHash -Algorithm SHA1 .\sample.doc`
* **Resultado:** `06727FFDA60359236A8029E0B3E8A0FD11C23313`.
* **Descoberta:** O VirusTotal classificou o arquivo como um `Office Open XML Document`.

> **📍 [INCLUIR PRINT 2: Janela do PowerShell mostrando o resultado do comando Get-FileHash]**

### Fase 2: Olhando "Debaixo do Capô" (Questões 3 e 4)
Como o documento é na verdade um arquivo compactado (XML), usei o **7-Zip** para explorar as pastas internas.
* **Onde achei o link:** No arquivo `word/_rels/document.xml.rels`.
* **URL Maliciosa:** `https://www.xmlformats.com/office/word/2022/wordprocessingDrawing/RDF842L.html!`.

> **📍 [INCLUIR PRINT 3: Editor de texto mostrando o código XML com o link malicioso destacado]**

### Fase 3: Pesquisa Técnica - OSINT (Questões 5, 8 e 9)
Nesta fase, utilizei técnicas de **OSINT (Open Source Intelligence)** consultando blogs de segurança (como Huntress e Elastic) para entender as regras do exploit.
* **Tamanho do Payload:** Descobri que o arquivo HTML remoto precisa de pelo menos **4096 bytes** de preenchimento (padding) para ativar o protocolo MSDT.
* **Classificação MITRE:** Identifiquei a técnica como **T1218** (System Binary Proxy Execution).
* **CVE:** Confirmado como **CVE-2022-30190**.

### Fase 4: Análise de Comportamento e Detecção (Questões 6 e 7)
Aqui o foco foi entender o que acontece no sistema quando o vírus "acorda".
* **Evasão:** O malware executa o comando `taskkill /f /im msdt.exe` para fechar a ferramenta de diagnóstico rapidamente antes que o usuário perceba algo errado.
* **Regra de Detecção (Event ID 4688):** Para detectar esse ataque em logs do Windows, monitorei a árvore de processos:
    * **Processo Filho:** `sdiagnhost.exe`
    * **Processo Pai:** `msdt.exe`

---

## 4. Conclusão do Aprendizado
Este laboratório foi fundamental para entender o conceito de **Living off the Land (LotL)**: usar ferramentas que já existem no Windows para fins maliciosos. Aprendi que a segurança não é apenas sobre antivírus, mas sobre monitorar comportamentos anômalos de programas confiáveis.

---
**Status do Laboratório:** 100% Concluído.

> **📍 [INCLUIR PRINT 4: Print final do site BTLO com as 9 questões marcadas como "Solved"]**
