# CPU Didática — Trabalho 2

Implementação de uma CPU simples inspirada na arquitetura RISC-V (modelo *load-store*), desenvolvida no **Logisim Evolution v4.0**. A CPU executa três instruções: `LOAD`, `STORE` e `ADD`.

---

## Integrantes

Pietro Rossi,
 Felipe Ballarin,
 Marcelo Silva,
 Murilo Guiot,
 André Barros.

---

## Estrutura do Repositório

```
.
├── modelo_banco_registradores.circ   # Arquivo principal do circuito (Logisim)
├── data_memory_teste                 # Imagem de memória de dados pré-carregada
├── instruction_memory_test           # Imagem de memória de instruções com o programa de teste
└── README.md
```

---

## Arquitetura

A CPU segue o modelo **load-store**: operações aritméticas ocorrem apenas entre registradores, e o acesso à memória é feito exclusivamente por `LOAD` e `STORE`.

### Componentes Implementados

| Componente | Descrição |
|---|---|
| **PC** | Registrador de 5 bits que armazena o endereço da próxima instrução |
| **Instruction Memory** | RAM 32×16 com leitura assíncrona; armazena o programa |
| **IR** | Instruction Register de 16 bits; captura a instrução atual |
| **Bloco de Registradores** | Quatro registradores de 16 bits: R0, R1, R2 e R3 |
| **ULA** | Somador de 16 bits |
| **Memória de Dados** | RAM 32×16 não-volátil; armazena os dados manipulados |
| **Unidade de Controle** | Decodifica o opcode e gera os sinais de controle |
| **clock_marcelo** | Subcircuito responsável pelo PC, IR e busca de instrução |
| **Memorias_IR_Murilo** | Subcircuito da memória de dados |

### Registradores

| Registrador | Comportamento |
|---|---|
| R0 | Sempre zero (hardwired) |
| R1, R2, R3 | Uso geral |

---

## Formato das Instruções (16 bits)

```
[ opcode ][ rd/rs2 ][ rs1 ][ rs2 ][ imediato ]
  15–14     13–12    11–10   9–8      7–0
  2 bits    2 bits   2 bits  2 bits   8 bits
```

### Opcodes

| Opcode | Instrução | Operação |
|---|---|---|
| `00` | `LOAD` | `Reg[rd] = Mem[Reg[rs1] + offset]` |
| `01` | `STORE` | `Mem[Reg[rs1] + offset] = Reg[rs2]` |
| `10` | `ADD` | `Reg[rd] = Reg[rs1] + Reg[rs2]` |
| `11` | — | Reservado |

---

## Programa de Teste

O programa carrega dois valores da memória, soma-os e armazena o resultado.

### Instruções

| Instrução | Hex | Comportamento esperado |
|---|---|---|
| `LOAD R1, 10(R0)` | `010A` | R1 = Mem[10] |
| `LOAD R2, 11(R0)` | `020B` | R2 = Mem[11] |
| `ADD R3, R1, R2` | `8D80` | R3 = R1 + R2 |
| `STORE R3, 12(R0)` | `430C` | Mem[12] = R3 |

### Dados iniciais na memória

| Endereço | Valor |
|---|---|
| `0x0A` (10) | `0x0007` (7) |
| `0x0B` (11) | `0x0005` (5) |

### Resultado esperado

Após 4 ciclos de clock:
- **R1 = 7**, **R2 = 5**, **R3 = 12**
- **Mem[12] = 0x000C** (12)

---

## Como Executar

### Pré-requisitos

- [Logisim Evolution v4.0](https://github.com/logisim-evolution/logisim-evolution/releases)
- Java 11 ou superior

### Passos

1. Abra o Logisim Evolution
2. Carregue o arquivo `modelo_banco_registradores.circ`
3. Carregue os dados na memória de instruções:
   - Expanda `clock_marcelo` → clique direito em `Instruction_Memory` → **Editar conteúdo**
   - Digite os valores: `010a 020b 8d80 430c` nos endereços 0, 1, 2 e 3
4. Carregue os dados na memória de dados:
   - Expanda `Memorias_IR_Murilo` → clique direito em `Data_Memory` → **Editar conteúdo**
   - Endereço `0A` = `0007`, endereço `0B` = `0005`
5. Selecione **CPU_Principal** na lista de circuitos
6. Pressione **Ctrl+R** para reiniciar
7. Pressione **Ctrl+K** para ativar o clock automático
8. Após alguns ciclos, pause e verifique:
   - `Data_Memory` endereço `0C` deve conter `000c`
   - Registradores R1=7, R2=5, R3=12 no `Bloco_Registradores`

---

## Sinais de Controle

| Sinal | Ativo em | Função |
|---|---|---|
| `Mem_to_Reg` | LOAD | Seleciona dado da memória para escrita no registrador |
| `MemWrite` | STORE | Habilita escrita na memória de dados |
| `Load` | LOAD | Indica leitura de memória |
| `ULA_Src` | LOAD / STORE | Seleciona imediato como segundo operando da ULA |

---

## Observações

- O registrador IR usa gatilho na **borda de descida** do clock para garantir que a instrução seja capturada corretamente após a atualização do PC
- A memória de dados é **não-volátil**, portanto os dados iniciais persistem entre simulações
- A memória de instruções usa **leitura assíncrona**, permitindo que a instrução fique disponível imediatamente após a mudança do PC
