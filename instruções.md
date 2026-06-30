# 🛣️ Sistema Inteligente de Gerenciamento para Pistas Piezoelétricas

Este repositório contém os submódulos e o circuito integrado final desenvolvidos no simulador **Wired Panda** (2026). O sistema calcula a potência gerada pelo tráfego rodoviário a partir de sensores piezoelétricos e realiza a tomada de decisão para armazenamento de energia usando lógica puramente combinatória.

---

## 📑 Índice dos Arquivos `.panda`

1. [Full Adder (`full_adder.panda`)](#1-full_adderpanda---somador-completo-de-1-bit)
2. [Somador de 4 Bits (`somador_4bits.panda`)](#2-somador_4bitspanda---somador-combinatorio-de-4-bits)
3. [Multiplicador Digital (`multiplicador.panda`)](#3-multiplicadorpanda---calculo-da-potencia-bruta)
4. [Comparador de Temperatura (`comparador_temperatura.panda`)](#4-comparador_temperaturapanda---validador-de-limiar-termico)
5. [Validador de Potência (`validacao_potencia.panda`)](#5-validacao_potenciapanda---filtro-de-eficiencia-energetica)
6. [Decodificador com Saturação (`display_saturado.panda`)](#6-display_saturadopanda---decodificador-customizado-com-saturacao-no-9)
7. [Sistema Final Integrado (`circuito_final.panda`)](#7-circuito_finalpanda---sistema-completo-integrado)

---

### 1. `full_adder.panda` - Somador Completo de 1 Bit

Módulo básico que realiza a soma aritmética de dois bits de entrada, considerando o transporte de entrada (*Carry-in*).

* **📥 Entradas:** `A` (Operando A), `B` (Operando B), `Cin` (*Carry-in*).
* **📤 Saídas:** `Sum` (Resultado da soma), `Cout` (*Carry-out*).
* **⚙️ Como Testar:** * Se apenas **uma** entrada for `1`: `Sum = 1` e `Cout = 0`.
  * Se **duas** entradas forem `1`: `Sum = 0` e `Cout = 1`.
  * Se as **três** entradas forem `1`: `Sum = 1` e `Cout = 1`.

---

### 2. `somador_4bits.panda` - Somador Combinatório de 4 Bits

Bloco aritmético estrutural construído a partir do cascateamento de 4 unidades do módulo *Full Adder*.

* **📥 Entradas:** `A3, A2, A1, A0` (Vetor A), `B3, B2, B1, B0` (Vetor B), `Cin` (Transporte inicial).
* **📤 Saídas:** `S3, S2, S1, S0` (Vetor Soma de 4 bits), `Cout` (Transporte final).
* **⚙️ Como Testar:** Defina `Cin=0`. Ative as chaves `A0=1` (A=1) e `B0=1` (B=1). As saídas devem indicar o valor decimal 2 (`S1=1`, demais em `0`).

---

### 3. `multiplicador.panda` - Cálculo da Potência Bruta

A Unidade Lógica Aritmética (ULA) do sistema. Realiza o produto de 4x4 bits utilizando uma matriz de portas AND para os produtos parciais, combinada a **três CIs somadores de 4 bits e quatro blocos Full Adders** estruturais para otimizar o tempo de propagação do sinal.

* **📥 Entradas:** `F3, F2, F1, F0` (Sensor de Força), `v3, v2, v1, v0` (Sensor de Velocidade).
* **📤 Saídas:** `P7` até `P0` (Vetor de 8 bits contendo o produto final $P = F \times v$).
* **⚙️ Como Testar:** * Ative as chaves para aplicar Força máxima e Velocidade máxima (F=15 e v=15). O barramento de saída deve indicar perfeitamente o binário de 225 decimal ($11100001_2$).

---

### 4. `comparador_temperatura.panda` - Validador de Limiar Térmico

Circuito discreto projetado para bloquear a operação caso a temperatura dos transdutores ultrapasse o limite seguro de 11 decimal ($1011_2$).

* **📥 Entradas:** `T3, T2, T1, T0` (Barramento do sensor de temperatura).
* **📤 Saídas:** `teste seguro` (`1` para temperatura operacional segura $\le 11$; `0` para estado crítico $> 11$).
* **⚙️ Como Testar:** * Configure as chaves para 10 ($1010_2$): a saída `teste seguro` ficará ativa (`1`).
  * Altere para 12 ($1100_2$): a saída deve ir imediatamente para `0`.

---

### 5. `validacao_potencia.panda` - Filtro de Eficiência Energética

Malha lógica combinatória que verifica se a potência calculada é estritamente maior que o limite de corte de 64 decimal ($01000000_2$).

* **📥 Entradas:** `P7, P6, P5, P4, P3, P2, P1, P0` (Barramento de 8 bits vindo do multiplicador).
* **📤 Saídas:** `P > 64` (`1` se válido, `0` se insuficiente).
* **⚙️ Como Testar:** * Ative apenas a chave `P6=1` (Potência exatamente igual a 64). A saída permanece em `0`.
  * Mantenha `P6=1` e ative `P0=1` (Potência = 65). A saída deve ligar (`1`).

---

### 6. `display_saturado.panda` - Decodificador Customizado com Saturação no 9

Circuito de interface visual projetado inteiramente do zero com portas lógicas discretas e inversores. Ele recebe os 4 bits mais significativos de potência e aciona diretamente um display de 7 segmentos comum. Possui uma malha de saturação booleana integrada para impedir que o display apresente falhas visuais ou apague caso a entrada assuma valores hexadecimais de 10 a 15 (`A` a `F`).

* **📥 Entradas:** `P7, P6, P5, P4` (Os 4 bits mais significativos de potência isolados, sendo P7 o MSB).
* **📤 Saídas:** Conexões diretas para os segmentos do display (`a, b, c, d, e, f, g`).
* **⚙️ Como Testar:**
  * **Operação Linear (0 a 9):** Configure as chaves com qualquer valor de 0 a 9 (ex: `P7=0, P6=1, P5=0, P4=1`, equivalente a 5). O display exibirá perfeitamente o caractere **5**.
  * **Atuação da Saturação (10 a 15):** Suba os valores além de 9 (ex: ative `P7=1, P6=1, P5=0, P4=0`, equivalente a 12). A rede de portas lógicas forçará internamente a combinação $1001_2$, fazendo com que o display trave e permaneça exibindo o dígito **9**, confirmando a saturação combinatória.

---

### 7. `circuito_final.panda` - Sistema Completo Integrado

A integração definitiva do projeto, unificando todos os módulos anteriores na mesma grade de simulação para gerar as duas saídas principais requisitadas.

* **📥 Entradas:** * `F[3:0]` (Sensor de Força - 4 bits)
  * `v[3:0]` (Sensor de Velocidade - 4 bits)
  * `T[3:0]` (Sensor de Temperatura - 4 bits)
  * `SENSOR DE PRESENÇA` (Chave digital - 1 bit)
* **📤 Saídas:**
  * **SAÍDA 1 (Display de 7 Segmentos):** Mostra o índice de potência instantânea de 0 a 9 baseado nos 4 MSBs ($P_7, P_6, P_5, P_4$) operando com a lógica de saturação descrita no item 6.
  * **SAÍDA 2 (`ATIVAR ARMAZENAMENTO`):** LED/Output pin indicador controlado por uma porta AND tripla de segurança. Só ativa em nível alto (`1`) se, simultaneamente: a potência for eficiente ($P > 64$), a temperatura estiver na faixa segura ($T \le 11$) e a presença do veículo for confirmada.

* **⚙️ Passo a Passo para Demonstração Prática:**
  1. Forneça condições ideais de geração: Força alta (ex: 12), Velocidade alta (ex: 8) e ligue a chave do `SENSOR DE PRESENÇA`. Mantenha a Temperatura baixa (ex: 4).
  2. Observe que o display exibirá o índice correspondente e o sinal `ATIVAR ARMAZENAMENTO` acenderá de imediato.
  3. Mantenha os estímulos e suba a temperatura alterando las chaves `T` para o valor 13 ($1101_2$). Note que o display de potência continua funcionando normalmente, mas o LED `ATIVAR ARMAZENAMENTO` apaga instantaneamente, comprovando a eficácia e robustez dos bloqueios de segurança do hardware combinatório.