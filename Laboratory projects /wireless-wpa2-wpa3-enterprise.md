# Wireless Security Lab — WPA2/WPA3 Enterprise

> **Laboratório educacional de segurança wireless**
>
> Todos os testes documentados neste projeto foram realizados exclusivamente em ambiente de laboratório controlado e autorizado, utilizando a infraestrutura disponibilizada para fins educacionais.

---

## 📌 Sobre o laboratório

Este documento registra a prática realizada no módulo de **Wireless Hacking**, com foco em redes **WPA2/WPA3
Enterprise**.

O objetivo foi compreender o processo de reconhecimento de uma rede corporativa wireless, analisar sua
autenticação, reproduzir um cenário de **Evil Twin** em laboratório e estudar a captura e posterior análise do
material de autenticação.

### Objetivos

- Identificar redes wireless e seus respectivos parâmetros.
- Compreender BSSID, ESSID, canais e bandas de frequência.
- Entender diferenças entre redes PSK e Enterprise.
- Analisar o funcionamento básico de autenticação WPA2/WPA3 Enterprise.
- Reproduzir um cenário de Evil Twin em laboratório.
- Observar um processo de desautenticação e reconexão.
- Analisar material de autenticação capturado.
- Utilizar Hashcat para recuperação de uma senha de laboratório.
- Identificar medidas de proteção contra esse tipo de cenário.

---

# 1. Reconhecimento das redes wireless

O primeiro comando utilizado foi:

```bash
airodump-ng wlan1
```

### O que é o `airodump-ng`?

O `airodump-ng` faz parte do conjunto **Aircrack-ng** e pode ser utilizado para monitorar redes wireless.

Entre as informações que podem ser observadas estão:

| Informação | Descrição |
|---|---|
| BSSID | Identificador do Access Point, normalmente seu endereço MAC|
| ESSID | Nome da rede Wi-Fi|
| Channel | Canal utilizado pela rede|
| Encryption | Mecanismo de criptografia|
| Authentication | Método de autenticação|
| STATION | Clientes wireless observados|

Inicialmente, a rede corporativa utilizada no laboratório não foi identificada.

Isso ocorreu porque a interface não estava monitorando adequadamente a banda utilizada pela rede alvo.

---

# 2. Monitoramento das bandas

Para ampliar o monitoramento, foi utilizado:

```bash
airodump-ng wlan1 --band ab
```

A opção `--band` permite selecionar as bandas que serão monitoradas.

Neste laboratório:

- `a` → banda de 5 GHz;
- `b` → banda de 2,4 GHz.

Portanto:

```text
--band ab
```

permite monitorar ambas as bandas.

Após essa alteração, a rede corporativa pôde ser identificada:

```text
ESSID: wifi-corp
BSSID: F0:9F:C2:71:22:1A
```

---

## 🔎 Termos importantes

### ESSID

**ESSID (Extended Service Set Identifier)** é o nome associado à rede wireless que normalmente é apresentado ao usuário.

Exemplo:

```text
wifi-corp
```

### BSSID

**BSSID (Basic Service Set Identifier)** identifica o ponto de acesso dentro da rede wireless. Em redes Wi-Fi
convencionais, normalmente corresponde ao endereço MAC da interface wireless do Access Point.

Exemplo:

```text
F0:9F:C2:71:22:1A
```

---

# 3. Monitoramento específico da rede

Depois de identificar a rede, o monitoramento foi direcionado ao ESSID:

```bash
airodump-ng wlan1 --band ab --essid wifi-corp
```

Isso reduziu a quantidade de informações apresentadas e facilitou a observação da rede utilizada no laboratório.

Foi necessário aguardar até que um dispositivo associado à rede realizasse comunicação.

O cliente observado no laboratório possuía o endereço:

```text
64:32:A8:BA:6C:41
```

> **Nota:** os endereços apresentados neste documento pertencem ao ambiente de laboratório utilizado no exercício.

---

# 4. WPA2/WPA3 Enterprise

Redes corporativas podem utilizar **WPA2/WPA3 Enterprise**, que possuem características diferentes das redes
domésticas baseadas em PSK.

### WPA2/WPA3-PSK

Utiliza uma chave pré-compartilhada:

```text
Wi-Fi
  ↓
Senha compartilhada
  ↓
Autenticação
```

### WPA2/WPA3 Enterprise

Normalmente utiliza mecanismos baseados em:

```text
Cliente
   ↓
802.1X / EAP
   ↓
Access Point
   ↓
Servidor de autenticação
   ↓
RADIUS
```

Nesse modelo, a autenticação pode ser individualizada para cada usuário.

---

# 5. Criação de um Evil Twin

A próxima etapa consistiu em reproduzir um cenário de **Evil Twin**.

Um Evil Twin é um Access Point que tenta se apresentar como uma rede legítima, normalmente utilizando o mesmo
ESSID ou características semelhantes.

No laboratório, foi utilizada a ferramenta **EAPHammer**.

A ferramenta foi obtida e instalada no ambiente Kali Linux utilizado no exercício.

---

# 6. Geração dos certificados

Antes de iniciar o Access Point de laboratório, foram gerados certificados utilizando:

```bash
python eaphammer --cert-wizard
```

O assistente solicita informações utilizadas na criação dos certificados.

Também foi executada a geração do material relacionado a **DH (Diffie-Hellman)**:

```bash
python eaphammer --cert-wizard dh
```

### Por que certificados são relevantes?

Em determinados métodos EAP, certificados digitais fazem parte do processo de autenticação e podem ser
utilizados pelo cliente para verificar a identidade do servidor.

Por isso, a validação correta do certificado é uma importante medida de segurança contra cenários de
infraestrutura wireless falsa.

---

# 7. Inicialização do Access Point de laboratório

Com os certificados preparados, o Access Point foi iniciado utilizando:

```bash
python eaphammer -i wlan6 --auth wpa-eap --creds --essid wifi-corp
```

### Parâmetros

| Parâmetro | Função |
|---|---|
| `-i wlan6` | Interface wireless utilizada |
| `--auth wpa-eap` | Define autenticação baseada em EAP |
| `--creds` | Habilita a funcionalidade relacionada à captura de credenciais |
| `--essid wifi-corp` | Define o ESSID anunciado |

O objetivo era fazer com que o Access Point de laboratório anunciasse o mesmo ESSID da rede legítima.

---

# 8. Verificação do Access Point falso

Em outro terminal, foi utilizado:

```bash
airodump-ng wlan1 --band ab --essid wifi-corp
```

A análise permitiu observar a rede legítima e o Access Point criado para o laboratório.

Esse é o cenário característico de um **Evil Twin**:

```text
                    wifi-corp
                       │
             ┌─────────┴─────────┐
             │                   │
       Access Point         Access Point
         legítimo               falso
             │                   │
             │                   │
          Clientes          laboratório
```

---

# 9. Desautenticação no laboratório

Para testar o comportamento do cliente diante da perda de associação, foi realizado um ataque de
desautenticação no ambiente controlado.

A ferramenta utilizada foi o `aireplay-ng`.

A estrutura utilizada foi:

```bash
aireplay-ng -0 15 -a F0:9F:C2:71:22:1A -c 64:32:A8:BA:6C:41 wlan1
```

### Parâmetros

| Parâmetro | Significado |
|---|---|
| `-0` | Modo de desautenticação |
| `15` | Quantidade de quadros enviados |
| `-a` | BSSID do Access Point |
| `-c` | MAC do cliente |
| `wlan1` | Interface utilizada |

O objetivo era fazer o cliente perder temporariamente sua associação com o Access Point legítimo e observar
sua tentativa de reconexão.

---

# 10. Ajuste do canal

Durante a execução ocorreu um erro relacionado ao canal utilizado pela interface.

Foi necessário configurar a interface `wlan1` para o mesmo canal utilizado pela rede do laboratório:

```bash
iwconfig wlan1 channel 44
```

O `iwconfig` permite consultar e configurar parâmetros de interfaces wireless no Linux.

Após o ajuste, o procedimento pôde ser executado normalmente.

---

# 11. Observação da autenticação

Após a desautenticação, o cliente iniciou novamente o processo de conexão.

Como o Access Point de laboratório anunciava o mesmo ESSID, foi possível observar uma nova tentativa de autenticação.

A EAPHammer capturou o material necessário para a etapa seguinte do exercício.

É importante destacar que, nesse cenário, o objetivo não é simplesmente obter uma senha em texto puro.

O material capturado representa informações relacionadas ao processo de autenticação, que podem posteriormente
ser submetidas a tentativas offline de recuperação da senha.

---

# 12. Preparação do material para análise

O material obtido foi disponibilizado em formato compatível com o Hashcat.

Foi criado um arquivo chamado `hash`:

```bash
nano hash
```

O conteúdo capturado foi inserido nesse arquivo.

### O que é o `nano`?

`nano` é um editor de texto simples executado diretamente no terminal Linux.

---

# 13. Recuperação da senha com Hashcat

Foi utilizado o Hashcat para realizar um ataque baseado em wordlist:

```bash
hashcat -a 0 -m 5500 hash ../kali/rockyou.txt
```

## Parâmetros

### `-a 0`

Define o **Attack Mode 0**, conhecido como *Straight Attack*.

Nesse modo, o Hashcat testa as palavras presentes em uma wordlist contra o material fornecido.

### `-m 5500`

Define o tipo/formato de hash que o Hashcat deve interpretar.

### `hash`

Arquivo contendo o material capturado durante o laboratório.

### `rockyou.txt`

Wordlist utilizada para realizar as tentativas.

---

# 14. RockYou

A `rockyou.txt` é uma wordlist amplamente utilizada em:

- laboratórios de segurança;
- CTFs;
- testes de força de senhas;
- demonstrações de password cracking.

Ela contém senhas provenientes de vazamentos históricos e, por isso, é frequentemente utilizada para
demonstrar ataques baseados em dicionário.

---

# 15. Resultado

Após a execução do Hashcat, foi possível recuperar a senha utilizada pelo usuário do laboratório:

```text
bulldogs1234
```

O resultado demonstrou que uma senha fraca pode ser suscetível a ataques de recuperação offline quando o
atacante consegue obter o material necessário para realizar as tentativas.

---

# 16. Conceitos praticados

Durante este laboratório foram trabalhados:

- Wireless Monitoring
- BSSID
- ESSID
- canais wireless
- bandas de 2,4 GHz e 5 GHz
- WPA2 Enterprise
- WPA3 Enterprise
- 802.1X
- EAP
- RADIUS
- Access Point
- Evil Twin
- certificados digitais
- Diffie-Hellman
- desautenticação
- autenticação wireless
- captura de material de autenticação
- ataques offline
- Hashcat
- wordlists
- RockYou

---

# 17. Principais riscos identificados

O laboratório demonstra que a segurança de uma rede Enterprise não depende apenas da utilização de WPA2 ou WPA3.

Também é necessário considerar:

### Validação de certificados

Clientes devem validar corretamente os certificados apresentados durante a autenticação EAP.

### Credenciais fortes

Senhas fracas podem ser mais suscetíveis a ataques offline baseados em listas de palavras (*wordlists*).

### AP Renegado (Rogue AP) / Evil Twin

A presença de Access Points não autorizados pode representar risco para usuários e para a infraestrutura corporativa.

### Monitoramento

Redes corporativas devem possuir mecanismos capazes de identificar comportamentos anômalos e Access Points não
autorizados.

---

# 18. Recomendações de segurança

Algumas medidas que podem reduzir os riscos observados são:

- utilizar métodos EAP adequadamente configurados;
- validar certificados no cliente;
- utilizar credenciais fortes;
- desabilitar métodos de autenticação desnecessários;
- monitorar Access Points não autorizados;
- implementar Wireless Intrusion Detection/Prevention (WIDS/WIPS) quando apropriado;
- segmentar redes corporativas;
- monitorar eventos de autenticação;
- manter a infraestrutura wireless atualizada.

---

# 19. Conclusão

Este laboratório permitiu compreender, de forma prática, o funcionamento de uma rede wireless corporativa e os
riscos associados a uma configuração inadequada de autenticação.

Além da utilização das ferramentas, o exercício ajudou a compreender a relação entre:

```text
Reconhecimento
      ↓
Identificação da infraestrutura
      ↓
Análise da autenticação
      ↓
Evil Twin
      ↓
Reconexão do cliente
      ↓
Captura do material de autenticação
      ↓
Análise offline
      ↓
Avaliação do risco
      ↓
Mitigação
```

A principal conclusão é que a segurança de redes wireless corporativas deve ser analisada de forma abrangente,
considerando **autenticação, certificados, credenciais, clientes, Access Points e mecanismos de
monitoramento**, e não apenas o algoritmo de criptografia utilizado.

---

## ⚠️ Aviso de uso

As técnicas apresentadas neste documento foram utilizadas exclusivamente para fins educacionais em ambiente de
laboratório controlado.

Não realize testes de desautenticação, captura de autenticação, Evil Twin ou recuperação de credenciais contra
redes, dispositivos ou usuários sem autorização explícita.
