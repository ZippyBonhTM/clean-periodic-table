# Requirements – ListAllChemicalElements

## Overview
O caso de uso deve retornar a lista de elementos químicos.

---
## Functional Requirements

### **FR1 — Listar elementos químicos**
* [ ] O sistema deve listar os elementos químicos em uma lista de elementos químicos.

### **FR2 — Propagar erros**
* [ ] Qualquer erro na operação fetch do repositório deve ser repassado ao chamador.

## Non-Functional Requirements

### **NFR1 — Independência**
* [ ] O caso de uso não deve conhecer detalhes concretos do repositório.

### **NFR2 — Estrutura de dados consistente**
* [ ] O retorno deve sempre ser uma lista de Elements.