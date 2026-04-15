# Aula 14/04/2026 - Componentização em React Native

---

## Objetivo da Aula

Na aula anterior evoluímos o projeto mobile integrando TypeScript ao React Native. Organizamos o domínio do sistema criando estruturas reutilizáveis de tipos e interfaces.

**O que criamos na aula passada:**

- Estrutura `src/types/` (`Especialidade`, `Paciente`, `StatusConsulta`)
- Estrutura `src/interfaces/` (`Medico`, `Consulta`)
- Modelagem completa do domínio do sistema
- Objetos tipados no `App.tsx`
- Estado tipado com `useState`
- Renderização condicional baseada em tipos

Hoje vamos evoluir a arquitetura do projeto aplicando um dos conceitos mais importantes do React: **componentização**.

Em vez de manter toda a interface dentro de um único arquivo gigante, começaremos a extrair partes da interface para componentes reutilizáveis e autossuficientes.

---

## Objetivos Específicos da Aula

- Criar a pasta `src/components/`
- Extrair o card de consulta para um componente próprio
- Criar props tipadas utilizando TypeScript
- Implementar callbacks para comunicação entre componentes
- Encapsular estilos dentro do componente
- Simplificar drasticamente o arquivo `App.tsx`
- Entender a diferença entre types locais e types globais

> ⚠️ **Importante:** Ainda não iremos trabalhar com navegação entre telas ou múltiplas páginas. Neste momento estamos construindo uma base arquitetural sólida para que o projeto possa crescer sem gerar código difícil de manter.

---

## Por Que Componentizar?

### O Problema

Na aula anterior, todo o JSX responsável por renderizar o card de consulta estava dentro do arquivo `App.tsx`. Isso funciona em projetos pequenos e didáticos, porém rapidamente gera problemas sérios de manutenção em projetos reais.

**Sintomas de código mal organizado:**

- Arquivo com centenas de linhas
- Muitas responsabilidades misturadas
- Estilos centralizados e confusos
- Difícil de debugar e testar
- Impossível de reutilizar código

### Antes da Refatoração

Estrutura conceitual do `App.tsx`:

```tsx
<View style={styles.container}>
  <ScrollView>
    <View style={styles.header}>
      {/* Cabeçalho */}
    </View>

    <View style={styles.card}>
      {/* 100+ LINHAS de JSX do card aqui! */}
      <View style={styles.statusBadge}>...</View>
      <View style={styles.secao}>...</View>
      <View style={styles.secao}>...</View>
      <View style={styles.acoes}>...</View>
    </View>

    <View style={styles.rodape}>
      {/* Rodapé */}
    </View>
  </ScrollView>
</View>
```

> ❌ **Problema:** O card da consulta está completamente misturado com a lógica do aplicativo.

### Depois da Refatoração

```tsx
<View style={styles.container}>
  <ScrollView>
    <View style={styles.header}>
      {/* Cabeçalho */}
    </View>

    {/* 1 linha! Limpo, elegante, reutilizável */}
    <ConsultaCard
      consulta={consulta}
      onConfirmar={confirmarConsulta}
      onCancelar={cancelarConsulta}
    />

    <View style={styles.rodape}>
      {/* Rodapé */}
    </View>
  </ScrollView>
</View>
```

> ✅ **Solução:** O card foi extraído para um componente próprio!

---

## Benefícios da Componentização

A componentização é um dos pilares da arquitetura React. Vamos entender por quê:

### 1. Reusabilidade

O mesmo componente pode ser utilizado em várias telas. Se tivermos uma tela que lista 10 consultas, usamos `<ConsultaCard>` 10 vezes!

### 2. Manutenção Simplificada

Caso seja necessário alterar o card de consulta, a modificação ocorre apenas no componente. **Mudou uma vez → mudou em todos os lugares que ele é usado!**

### 3. Código Mais Organizado

Cada arquivo passa a ter uma responsabilidade específica:

- `App.tsx` → Layout geral da tela
- `ConsultaCard.tsx` → Renderização de uma consulta
- `Header.tsx` → Cabeçalho (futuro exercício!)

### 4. Legibilidade

Arquivos menores são infinitamente mais fáceis de compreender. Compare 200 linhas misturadas vs 3 arquivos de 70 linhas cada!

### 5. Testabilidade

Componentes isolados são muito mais simples de testar. Você pode testar `ConsultaCard` sem precisar iniciar o app inteiro!

---

## Nova Estrutura do Projeto

A partir desta aula criamos uma nova pasta para armazenar componentes:

```
src/
│
├── components/          ← NOVO! Componentes reutilizáveis
│   ├── ConsultaCard.tsx
│   └── index.ts         (exportação centralizada)
│
├── interfaces/          ← Types/Interfaces GLOBAIS
│   ├── consulta.ts      (usada em vários lugares)
│   └── medico.ts        (usada em vários lugares)
│
└── types/               ← Types GLOBAIS
    ├── especialidade.ts (usada em vários lugares)
    ├── paciente.ts      (usada em vários lugares)
    └── statusConsulta.ts (usada em vários lugares)
```

---

## Types Locais vs Types Globais — A Distinção FUNDAMENTAL

Este é um dos conceitos mais importantes sobre organização de código em TypeScript!

### Types/Interfaces GLOBAIS

**Localização:** `src/types/` ou `src/interfaces/`

**Quando usar:**
- Usados em múltiplos componentes/telas/services
- Representam o domínio da aplicação
- São as "entidades" do sistema

**Exemplos do nosso projeto:**

```typescript
// src/interfaces/consulta.ts
export interface Consulta {
  id: string;
  data: Date;
  medico: Medico;     // ← usado em vários lugares
  paciente: Paciente; // ← usado em vários lugares
  // ...
}

// src/types/paciente.ts
export type Paciente = {
  nome: string;
  cpf: string;
  // ...
};
```

> **Por que são globais?** `Consulta` é usada em: `ConsultaCard`, telas de listagem, services de API, etc. `Medico` é usada em: várias telas, formulários, serviços, etc. `Paciente` idem!

### Types LOCAIS

**Localização:** Dentro do próprio arquivo do componente

**Quando usar:**
- Usado em um único lugar
- Define a interface (props) de um componente
- É específico demais para ser global

**Exemplo do nosso projeto:**

```typescript
// src/components/ConsultaCard.tsx

// Este type SÓ existe aqui!
type ConsultaCardProps = {
  consulta: Consulta;   // ← Consulta vem de src/interfaces (global)
  onConfirmar?: () => void;
  onCancelar?: () => void;
};

export default function ConsultaCard(props: ConsultaCardProps) {
  // ...
}
```

> **Por que é local?** `ConsultaCardProps` é usada APENAS no componente `ConsultaCard`. Se apagarmos `ConsultaCard.tsx`, esse type desaparece junto. Não faz sentido poluir `src/types/` com isso!

### Comparação Visual

```
┌─────────────────────────────────────────────┐
│  src/interfaces/consulta.ts (GLOBAL)        │
│  ↓                                          │
│  export interface Consulta { ... }          │
│  ↓                                          │
│  Usada em:                                  │
│  • ConsultaCard.tsx                         │
│  • ConsultaList.tsx                         │
│  • AgendamentoScreen.tsx                    │
│  • HistoricoScreen.tsx                      │
│  • ConsultaService.ts                       │
│  • 10+ outros lugares                       │
└─────────────────────────────────────────────┘

┌─────────────────────────────────────────────┐
│  src/components/ConsultaCard.tsx (LOCAL)    │
│  ↓                                          │
│  type ConsultaCardProps = { ... }           │
│  ↓                                          │
│  Usada em:                                  │
│  • ConsultaCard.tsx ← APENAS AQUI!          │
└─────────────────────────────────────────────┘
```

### Regra Prática para Decidir

> "Este type será usado em mais de um arquivo?"
>
> **SIM** → `src/types/` ou `src/interfaces/`
> **NÃO** → Dentro do próprio arquivo

**Tabela de Decisão:**

| Type | Onde Vai? | Por Quê? |
|---|---|---|
| `Consulta` | `src/interfaces/` | Usada em 10+ lugares |
| `Medico` | `src/interfaces/` | Usada em 10+ lugares |
| `ConsultaCardProps` | `ConsultaCard.tsx` | Usada em 1 lugar |
| `HeaderProps` | `Header.tsx` | Usada em 1 lugar |
| `Especialidade` | `src/types/` | Usada em 5+ lugares |
| `ButtonVariant` (futuro) | `Button.tsx` | Usada em 1 lugar |

---

## IMPLEMENTAÇÃO - Criando o Componente Passo a Passo

Agora que entendemos a teoria, vamos para a prática!

### Passo 1: Criar o Arquivo

Crie: `src/components/ConsultaCard.tsx`

### Passo 2: Estrutura Básica

```tsx
/**
 * ConsultaCard - Componente Reutilizável
 *
 * Responsabilidade: Exibir os dados de UMA consulta
 */

import React from "react";
import { View, Text, StyleSheet, Button } from "react-native";
import { Consulta } from "../interfaces/consulta"; // ← import do type GLOBAL

// Type LOCAL (usado apenas aqui)
type ConsultaCardProps = {
  consulta: Consulta;
  onConfirmar?: () => void;
  onCancelar?: () => void;
};

export default function ConsultaCard({
  consulta,
  onConfirmar,
  onCancelar,
}: ConsultaCardProps) {

  return (
    <View style={styles.card}>
      <Text>{consulta.paciente.nome}</Text>
    </View>
  );
}

// Estilos LOCAIS (encapsulados no componente)
const styles = StyleSheet.create({
  card: {
    backgroundColor: "#fff",
    padding: 20,
    borderRadius: 16,
  },
});
```

### Passo 3: Adicionar Funções Auxiliares

Queremos formatar data e valor. Onde essas funções ficam?

**Resposta:** Dentro do componente! São auxiliares locais.

```tsx
export default function ConsultaCard({
  consulta,
  onConfirmar,
  onCancelar,
}: ConsultaCardProps) {

  // Auxiliar LOCAL - formata valor para R$ 150,00
  function formatarValor(valor: number): string {
    return valor.toLocaleString("pt-BR", {
      style: "currency",
      currency: "BRL",
    });
  }

  // Auxiliar LOCAL - formata data para 25/03/2026
  function formatarData(data: Date): string {
    return data.toLocaleDateString("pt-BR");
  }

  return (
    <View style={styles.card}>
      <Text>{consulta.paciente.nome}</Text>
      <Text>{formatarData(consulta.data)}</Text>
      <Text>{formatarValor(consulta.valor)}</Text>
    </View>
  );
}
```

> **Nota:** Se essas funções fossem usadas em vários componentes, criaríamos `src/utils/formatadores.ts`. Mas como são específicas deste card, ficam aqui!

---

### Passo 4: Props Opcionais

```typescript
type ConsultaCardProps = {
  consulta: Consulta;          // OBRIGATÓRIA
  onConfirmar?: () => void;    // OPCIONAL
  onCancelar?: () => void;     // OPCIONAL
};
```

**Por que opcional?**

**Cenário 1:** Tela de consultas agendadas

```tsx
<ConsultaCard
  consulta={consulta}
  onConfirmar={confirmar}
  onCancelar={cancelar}
/>
```

> Mostra os botões! Usuário pode interagir.

**Cenário 2:** Tela de histórico (consultas antigas)

```tsx
<ConsultaCard
  consulta={consultaAntiga}
/>
```

> Apenas visualização, sem botões!

---

## Fluxo de Comunicação: Pai ⇒ Filho ⇒ Pai

Este é um conceito **FUNDAMENTAL** do React.

**O Princípio:** O estado vive no PAI. O filho apenas COMUNICA mudanças através de callbacks. Isso mantém o fluxo unidirecional de dados do React.

### Diagrama do Fluxo

```
┌──────────────────────────────────────┐
│         App.tsx (PAI)                │
│                                      │
│  const [consulta, setConsulta] =     │
│    useState<Consulta>({...})         │  ← ESTADO AQUI
│                                      │
│  function confirmar() {              │
│    setConsulta({                     │
│      ...consulta,                    │
│      status: "confirmada"            │
│    });                               │
│  }                                   │
│                                      │
└──────────┬───────────────────────────┘
           │
           │  passa props ↓
           │  • consulta
           │  • onConfirmar={confirmar}
           │  • onCancelar={cancelar}
           │
┌──────────▼───────────────────────────┐
│    ConsultaCard.tsx (FILHO)          │
│                                      │
│  function ConsultaCard({             │
│    consulta,                         │
│    onConfirmar,                      │  ← RECEBE PROPS
│    onCancelar                        │
│  }) {                                │
│                                      │
│    return (                          │
│      <Button                         │
│        onPress={onConfirmar}         │  ← CHAMA CALLBACK
│      />                              │
│    );                                │
│  }                                   │
│                                      │
└──────────────────────────────────────┘
```

### Fluxo Completo (Passo a Passo)

1. Usuário clica no botão "Confirmar" no `ConsultaCard`
2. `ConsultaCard` chama `onConfirmar()` (função recebida via props)
3. A função executa no componente PAI (`App.tsx`)
4. O pai atualiza o estado: `setConsulta({ ...consulta, status: "confirmada" })`
5. React detecta mudança de estado no pai
6. React re-renderiza `App.tsx`
7. `App.tsx` passa a nova `consulta` atualizada para `ConsultaCard`
8. React re-renderiza `ConsultaCard` com novos dados
9. Usuário vê o status atualizado na tela!

---

## Código Completo

### `App.tsx` (Pai)

```tsx
/**
 * App.tsx - Aplicativo de Consultas Médicas
 * Versão 3: Componentização
 *
 * Evolução:
 * Aula 1 (31/03) → MVP Simples
 * Aula 2 (07/04) → Integração TypeScript
 * Aula 3 (14/04) → Componentização ← VOCÊ ESTÁ AQUI
 */

import React, { useState } from "react";
import { View, Text, StyleSheet, ScrollView } from "react-native";
import { StatusBar } from "expo-status-bar";

// Importando a modelagem TypeScript
import { Especialidade } from "./src/types/especialidade";
import { Paciente } from "./src/types/paciente";
import { Medico } from "./src/interfaces/medico";
import { Consulta } from "./src/interfaces/consulta";

// Importando o componente reutilizável
import { ConsultaCard } from "./src/components";

export default function App() {
  // Dados base (simulando o que tínhamos no backend)
  const cardiologia: Especialidade = {
    id: 1,
    nome: "Cardiologia",
    descricao: "Cuidados com o coração",
  };

  const medico1: Medico = {
    id: 1,
    nome: "Dr. Roberto Silva",
    crm: "CRM12345",
    especialidade: cardiologia,
    ativo: true,
  };

  const paciente1: Paciente = {
    id: 1,
    nome: "Carlos Andrade",
    cpf: "123.456.789-00",
    email: "carlos@email.com",
    telefone: "(11) 98765-4321",
  };

  // Estado da consulta
  const [consulta, setConsulta] = useState<Consulta>({
    id: 1,
    medico: medico1,
    paciente: paciente1,
    data: new Date(2026, 2, 10), // 10/03/2026
    valor: 350,
    status: "agendada",
    observacoes: "Consulta de rotina",
  });

  /**
   * Funções para manipular a consulta
   *
   * Essas funções serão passadas como props para o componente.
   * O componente não altera o estado diretamente - ele apenas
   * "comunica" ao pai (App) que uma ação foi solicitada.
   */
  function confirmarConsulta() {
    setConsulta({
      ...consulta,
      status: "confirmada",
    });
  }

  function cancelarConsulta() {
    setConsulta({
      ...consulta,
      status: "cancelada",
    });
  }

  return (
    <View style={styles.container}>
      <StatusBar style="light" />

      <ScrollView contentContainerStyle={styles.scrollContent}>
        {/* Cabeçalho */}
        <View style={styles.header}>
          <Text style={styles.titulo}>Sistema de Consultas</Text>
          <Text style={styles.subtitulo}>Consulta #{consulta.id}</Text>
        </View>

        {/*
          Componente ConsultaCard

          Veja como ficou mais simples!
          Antes: ~100 linhas de JSX no App.tsx
          Agora: 1 componente reutilizável

          Props:
          - consulta: objeto com todos os dados
          - onConfirmar: função a ser chamada ao confirmar
          - onCancelar: função a ser chamada ao cancelar
        */}
        <ConsultaCard
          consulta={consulta}
          onConfirmar={confirmarConsulta}
          onCancelar={cancelarConsulta}
        />

      </ScrollView>
    </View>
  );
}

/**
 * Estilos do App
 *
 * Note que removemos TODOS os estilos do card!
 * Eles agora estão encapsulados no componente ConsultaCard.
 *
 * App.tsx agora só tem estilos de layout geral:
 * - Container principal
 * - Cabeçalho
 * - Rodapé
 */
const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: "#79059C",
  },
  scrollContent: {
    padding: 20,
    paddingTop: 60,
  },
  header: {
    alignItems: "center",
    marginBottom: 24,
  },
  titulo: {
    fontSize: 28,
    fontWeight: "bold",
    color: "#fff",
    marginBottom: 8,
  },
  subtitulo: {
    fontSize: 18,
    color: "#fff",
    opacity: 0.9,
  },
  rodape: {
    marginTop: 24,
    padding: 16,
    backgroundColor: "rgba(255, 255, 255, 0.1)",
    borderRadius: 12,
  },
  rodapeTexto: {
    fontSize: 12,
    color: "#fff",
    textAlign: "center",
    lineHeight: 18,
    marginBottom: 4,
  },
});
```

### `ConsultaCard.tsx` (Filho)

```tsx
/**
 * =============================================================================
 * COMPONENTE: ConsultaCard
 * =============================================================================
 *
 * Este é nosso primeiro componente extraído!
 *
 * O que este componente faz?
 * → Exibe os dados de UMA consulta médica de forma organizada
 *
 * Por que criamos este componente?
 * → Reutilização: Se tivermos 10 consultas, usamos este componente 10 vezes
 * → Organização: App.tsx não precisa saber COMO renderizar um card
 * → Manutenção: Mudanças no visual do card acontecem apenas aqui
 * → Testabilidade: Podemos testar este componente isoladamente
 *
 * =============================================================================
 */

import React from "react";
import { View, Text, StyleSheet, Button } from "react-native";

// Importamos a interface Consulta que criamos na aula passada
// Ela vem de src/interfaces/ porque é usada em VÁRIOS lugares
import { Consulta } from "../interfaces/consulta";

/**
 * =============================================================================
 * TIPAGEM DAS PROPS (TYPE LOCAL DO COMPONENTE)
 * =============================================================================
 *
 * Atenção para esta distinção IMPORTANTE:
 *
 * Consulta → está em src/interfaces/ (usada em vários lugares)
 * ConsultaCardProps → está AQUI (usada APENAS neste componente)
 *
 * REGRA DE OURO:
 * Type/Interface usado em VÁRIOS componentes → src/types/ ou src/interfaces/
 * Type usado em UM componente só → dentro do próprio arquivo
 *
 * =============================================================================
 */
type ConsultaCardProps = {
  // A consulta que queremos exibir (OBRIGATÓRIA)
  consulta: Consulta;

  // Função chamada quando o usuário clica em "Confirmar" (OPCIONAL)
  // Por que opcional? Às vezes queremos só exibir, sem botões de ação!
  onConfirmar?: () => void;

  // Função chamada quando o usuário clica em "Cancelar" (OPCIONAL)
  onCancelar?: () => void;
};

/**
 * =============================================================================
 * COMPONENTE PRINCIPAL
 * =============================================================================
 *
 * Aqui usamos destructuring nas props - é uma técnica moderna do JavaScript
 *
 * Em vez de: function ConsultaCard(props) { const consulta = props.consulta; }
 * Fazemos: function ConsultaCard({ consulta, onConfirmar, onCancelar })
 *
 * Fica mais limpo e direto!
 *
 * =============================================================================
 */
export default function ConsultaCard({
  consulta,
  onConfirmar,
  onCancelar,
}: ConsultaCardProps) {

  /**
   * ===========================================================================
   * FUNÇÕES AUXILIARES (LOCAIS DO COMPONENTE)
   * ===========================================================================
   *
   * Estas funções existem APENAS para ajudar este componente.
   * Por isso ficam aqui dentro, não precisam estar em outro arquivo.
   *
   * Se fossem usadas em vários componentes, criaríamos:
   * src/utils/formatadores.ts
   *
   * ===========================================================================
   */

  // Formata um número para moeda brasileira (R$ 150,00)
  function formatarValor(valor: number): string {
    return valor.toLocaleString("pt-BR", {
      style: "currency",
      currency: "BRL",
    });
  }

  // Formata uma data no padrão brasileiro (25/03/2026)
  function formatarData(data: Date): string {
    return data.toLocaleDateString("pt-BR");
  }

  return (
    <View style={styles.card}>

      {/*
        -----------------------------------------------------------------------
        BADGE DO STATUS
        -----------------------------------------------------------------------
        Renderização condicional de estilos!

        Se status === "confirmada" → aplica styles.statusConfirmada (verde)
        Se status === "cancelada"  → aplica styles.statusCancelada (vermelho)
        Se status === "agendada"   → só o estilo padrão (roxo)
        -----------------------------------------------------------------------
      */}
      <View
        style={[
          styles.statusBadge,
          consulta.status === "confirmada" && styles.statusConfirmada,
          consulta.status === "cancelada" && styles.statusCancelada,
        ]}
      >
        <Text style={styles.statusTexto}>
          {consulta.status.toUpperCase()}
        </Text>
      </View>

      {/*
        -----------------------------------------------------------------------
        SEÇÃO: MÉDICO
        -----------------------------------------------------------------------
        Exibimos todas as informações do médico.
        Repare que acessamos: consulta.medico.nome, consulta.medico.crm, etc.
        Isso funciona porque tipamos tudo com TypeScript!
        -----------------------------------------------------------------------
      */}
      <View style={styles.secao}>
        <Text style={styles.label}>👨‍⚕️ Médico</Text>
        <Text style={styles.valor}>{consulta.medico.nome}</Text>
        <Text style={styles.info}>CRM: {consulta.medico.crm}</Text>
        <Text style={styles.info}>{consulta.medico.especialidade.nome}</Text>
      </View>

      {/*
        -----------------------------------------------------------------------
        SEÇÃO: PACIENTE
        -----------------------------------------------------------------------
        Repare que telefone é OPCIONAL na interface Paciente!
        Se não existir, não renderizamos nada.
        Isso é renderização condicional baseada em dados opcionais.
        -----------------------------------------------------------------------
      */}
      <View style={styles.secao}>
        <Text style={styles.label}>👤 Paciente</Text>
        <Text style={styles.valor}>{consulta.paciente.nome}</Text>
        <Text style={styles.info}>CPF: {consulta.paciente.cpf}</Text>
        <Text style={styles.info}>Email: {consulta.paciente.email}</Text>
        {consulta.paciente.telefone && (
          <Text style={styles.info}>Tel: {consulta.paciente.telefone}</Text>
        )}
      </View>

      {/*
        -----------------------------------------------------------------------
        SEÇÃO: DADOS DA CONSULTA
        -----------------------------------------------------------------------
        Aqui usamos as funções auxiliares formatarData() e formatarValor()

        Em vez de:
        <Text>{consulta.data.toLocaleDateString("pt-BR")}</Text>

        Fazemos:
        <Text>{formatarData(consulta.data)}</Text>

        Fica mais legível e fácil de manter!
        -----------------------------------------------------------------------
      */}
      <View style={styles.secao}>
        <Text style={styles.label}>📅 Dados da Consulta</Text>
        <Text style={styles.valor}>Data: {formatarData(consulta.data)}</Text>
        <Text style={styles.valor}>
          Valor: {formatarValor(consulta.valor)}
        </Text>
        {consulta.observacoes && (
          <Text style={styles.observacoes}>{consulta.observacoes}</Text>
        )}
      </View>

      {/*
        -----------------------------------------------------------------------
        BOTÕES DE AÇÃO (PROPS OPCIONAIS + CALLBACKS)
        -----------------------------------------------------------------------
        CONCEITO MUITO IMPORTANTE!

        Este componente NÃO gerencia o estado da consulta.
        Quem gerencia é o componente PAI (App.tsx).

        Renderização condicional em DOIS níveis:
        Nível 1: consulta.status === "agendada"
        → Só mostra botões se a consulta ainda estiver agendada

        Nível 2: onConfirmar && <Botao>
        → Só mostra o botão se a prop foi passada
        -----------------------------------------------------------------------
      */}
      <View style={styles.acoes}>
        {consulta.status === "agendada" && (
          <>
            {onConfirmar && (
              <View style={styles.botaoContainer}>
                <Button
                  title="Confirmar Consulta"
                  onPress={onConfirmar}
                  color="#4CAF50"
                />
              </View>
            )}
            {onCancelar && (
              <View style={styles.botaoContainer}>
                <Button
                  title="Cancelar Consulta"
                  onPress={onCancelar}
                  color="#F44336"
                />
              </View>
            )}
          </>
        )}

        {consulta.status === "confirmada" && (
          <View style={styles.mensagem}>
            <Text style={styles.mensagemTexto}>
              ✓ Consulta confirmada com sucesso!
            </Text>
          </View>
        )}

        {consulta.status === "cancelada" && (
          <View style={styles.mensagemCancelada}>
            <Text style={styles.mensagemTexto}>✗ Consulta cancelada</Text>
          </View>
        )}
      </View>
    </View>
  );
}

/**
 * =============================================================================
 * ESTILOS DO COMPONENTE (ENCAPSULADOS)
 * =============================================================================
 *
 * Todos os estilos relacionados ao card ficam AQUI, dentro do componente.
 *
 * Antes da componentização:
 * - App.tsx tinha ~20 estilos misturados
 * - Estilos do card + estilos do app tudo junto
 *
 * Depois da componentização:
 * - App.tsx tem só estilos de layout geral (container, header, footer)
 * - ConsultaCard.tsx tem só estilos do card
 * - Cada um cuida do seu!
 *
 * Isso é ENCAPSULAMENTO na prática.
 * O componente é AUTOSSUFICIENTE: tem seu JSX, sua lógica E seus estilos.
 *
 * =============================================================================
 */
const styles = StyleSheet.create({
  // Container principal do card
  card: {
    backgroundColor: "#fff",
    borderRadius: 16,
    padding: 20,
    // Sombra no iOS
    shadowColor: "#000",
    shadowOffset: { width: 0, height: 4 },
    shadowOpacity: 0.2,
    shadowRadius: 8,
    // Sombra no Android
    elevation: 5,
  },

  // Badge de status (agendada, confirmada, cancelada)
  statusBadge: {
    backgroundColor: "#FFA500", // Laranja (padrão para "agendada")
    alignSelf: "flex-start",
    paddingHorizontal: 16,
    paddingVertical: 8,
    borderRadius: 20,
    marginBottom: 20,
  },
  statusConfirmada: {
    backgroundColor: "#4CAF50", // Verde
  },
  statusCancelada: {
    backgroundColor: "#F44336", // Vermelho
  },
  statusTexto: {
    color: "#fff",
    fontWeight: "bold",
    fontSize: 12,
  },

  // Seções do card (médico, paciente, dados)
  secao: {
    marginBottom: 20,
    paddingBottom: 20,
    borderBottomWidth: 1,
    borderBottomColor: "#e0e0e0",
  },

  // Labels das seções (👨‍⚕️ Médico, 👤 Paciente, etc)
  label: {
    fontSize: 16,
    fontWeight: "bold",
    color: "#79059C",
    marginBottom: 8,
  },

  // Valores exibidos (nome do médico, nome do paciente, etc)
  valor: {
    fontSize: 18,
    color: "#333",
    marginBottom: 4,
  },

  // Informações complementares (CRM, CPF, email, etc)
  info: {
    fontSize: 14,
    color: "#666",
    marginBottom: 2,
  },

  // Observações (texto em itálico)
  observacoes: {
    fontSize: 14,
    color: "#555",
    fontStyle: "italic",
    marginTop: 8,
  },

  // Container das ações (botões e mensagens)
  acoes: {
    marginTop: 10,
  },

  // Espaçamento entre botões
  botaoContainer: {
    marginBottom: 12,
  },

  // Mensagem de sucesso (verde)
  mensagem: {
    backgroundColor: "#E8F5E9",
    padding: 16,
    borderRadius: 8,
    borderLeftWidth: 4,
    borderLeftColor: "#4CAF50",
  },

  // Mensagem de cancelamento (vermelho)
  mensagemCancelada: {
    backgroundColor: "#FFEBEE",
    padding: 16,
    borderRadius: 8,
    borderLeftWidth: 4,
    borderLeftColor: "#F44336",
  },
  mensagemTexto: {
    fontSize: 16,
    color: "#333",
    fontWeight: "600",
    textAlign: "center",
  },
});
```

### Por que NÃO colocar o estado no filho?

❌ **Errado:**

```tsx
// ❌ NÃO FAÇA ISSO!
function ConsultaCard({ consulta }: ConsultaCardProps) {
  const [localConsulta, setLocalConsulta] = useState(consulta);

  function confirmar() {
    setLocalConsulta({ ...localConsulta, status: "confirmada" });
  }
  // ...
}
```

**Problemas:**

- O pai não sabe que o estado mudou
- Outros componentes não recebem a atualização
- Dificulta sincronização de dados
- Quebra o fluxo unidirecional do React

✅ **Certo:**

```tsx
// ✅ FAÇA ASSIM!
function ConsultaCard({ consulta, onConfirmar }: ConsultaCardProps) {
  return (
    <Button onPress={onConfirmar} />  {/* ← Delega ao pai! */}
  );
}
```

---

## Encapsulamento de Estilos

Outro benefício GIGANTE da componentização!

### Antes: Estilos Misturados

```typescript
// App.tsx tinha TODOS os estilos misturados

const styles = StyleSheet.create({
  // Estilos do App
  container: { ... },
  header: { ... },

  // Estilos do Card MISTURADO!
  card: { ... },
  statusBadge: { ... },
  secao: { ... },
  label: { ... },
  valor: { ... },
  info: { ... },
  acoes: { ... },
  botaoContainer: { ... },

  // Estilos do App novamente
  rodape: { ... },
  rodapeTexto: { ... },
});
```

**Problemas:**
- Arquivo gigante (~20 estilos misturados)
- Difícil saber o que pertence a quê
- Mudanças no card afetam visualmente o `App.tsx`
- Nomes podem colidir

### Depois: Estilos Encapsulados

`App.tsx` — apenas estilos relacionados ao layout geral:

```typescript
const styles = StyleSheet.create({
  container: { ... },
  scroll: { ... },
  header: { ... },
  titulo: { ... },
  subtitulo: { ... },
  rodape: { ... },
  rodapeTexto: { ... },
});
```

`ConsultaCard.tsx` - apenas estilos relacionados ao card:

```typescript
const styles = StyleSheet.create({
  card: { ... },
  statusBadge: { ... },
  statusConfirmada: { ... },
  statusCancelada: { ... },
  secao: { ... },
  label: { ... },
  valor: { ... },
  info: { ... },
  observacoes: { ... },
  acoes: { ... },
  botaoContainer: { ... },
  mensagem: { ... },
  mensagemCancelada: { ... },
  mensagemTexto: { ... },
});
```

**Benefícios:**
- **Clareza:** Cada componente tem seus estilos
- **Manutenção:** Mudança visual do card → mexe só no `ConsultaCard.tsx`
- **Sem colisões:** Posso ter `styles.container` no App e no Card sem conflito!
- **Autossuficiência:** O componente carrega tudo que precisa (JSX + estilos)

---

## Exportação Centralizada (index.ts)

Para finalizar a componentização com chave de ouro, criamos um **barrel export**.

Arquivo: `src/components/index.ts`

```typescript
/**
 * Exportação centralizada dos componentes
 *
 * Permite importar assim:
 * import { ConsultaCard } from './src/components';
 *
 * Em vez de:
 * import ConsultaCard from './src/components/ConsultaCard';
 */

export { default as ConsultaCard } from "./ConsultaCard";

/**
 * Conforme criarmos mais componentes, adicionaremos aqui:
 *
 * export { default as Loading } from "./Loading";
 * export { default as EmptyState } from "./EmptyState";
 * export { default as Header } from "./Header";
 */
```

### Por Quê Import/Export Assim?

Sem `index.ts`:

```typescript
// Em App.tsx
import ConsultaCard from "./src/components/ConsultaCard";
import Header from "./src/components/Header";
import ConsultaList from "./src/components/ConsultaList";
```

Com `index.ts` (barrel export):

```typescript
// Em App.tsx
import { ConsultaCard, Header, ConsultaList } from "./src/components";
```

**Vantagens:**
- Imports mais limpos
- Centralização dos exports
- Padrão usado em projetos profissionais
- IntelliSense melhor no editor

---

## Comparação Completa: Antes vs Depois

### Métricas de Melhoria

| Métrica | Antes | Depois | Melhoria |
|---|---|---|---|
| Linhas em `App.tsx` | ~200 | ~120 | -40% |
| Estilos em `App.tsx` | ~20 | ~6 | -70% |
| Responsabilidades | Layout + Card + Estado | Layout + Estado | +Separação |
| Reusabilidade | 0% | 100% | ∞ |
| Testabilidade | Difícil | Fácil | ++++ |

---

## Quando Componentizar? (Guia Prático)

### Componentize Quando:

- **Reusabilidade:** O código é usado em múltiplos lugares
- **Tamanho:** Um bloco de JSX tem mais de 50 linhas
- **Responsabilidade Única:** O bloco tem uma função específica bem definida
- **Independência:** A lógica é independente do contexto
- **Testabilidade:** Você quer testar isoladamente

### NÃO Componentize Quando:

- **Uso Único Simples:** É usado uma única vez E é simples (< 20 linhas)
- **Forte Acoplamento:** Tem dependências fortes com o componente pai
- **Muito Pequeno:** É trivial demais (ex: 5 linhas)
- **Complica Mais:** A componentização piora a legibilidade

> **Regra de Ouro:** Componentize para MELHORAR o código, não por obrigação.

---

## Conceitos Aplicados na Aula (Checklist)

### TypeScript
- [x] Props tipadas com `type`
- [x] Optional properties (`?`)
- [x] Callback types (`() => void`)
- [x] Destructuring com tipos
- [x] Diferença entre types locais e globais
- [x] Import/export de interfaces

### React Native
- [x] Componente funcional
- [x] Props (passagem de dados pai → filho)
- [x] Callbacks (comunicação filho → pai)
- [x] Renderização condicional
- [x] Encapsulamento de estilos
- [x] Estado com `useState`

### Arquitetura
- [x] Separação de responsabilidades (SRP)
- [x] Reusabilidade
- [x] Encapsulamento
- [x] Fluxo unidirecional de dados
- [x] Organização de pastas
- [x] Barrel exports

---

## Evolução Completa do Projeto

```
AULA 1 (31/03/2026) — MVP Inicial
├── App.tsx (tudo em um arquivo)
├── useState básico
└── Interface funcional

AULA 2 (06/04/2026) — TypeScript Integrado
├── src/types/
├── src/interfaces/
├── Modelagem completa do domínio
└── Objetos tipados no App.tsx

AULA 3 (14/04/2026) — Componentização ← VOCÊ ESTÁ AQUI!
├── src/components/
│   ├── ConsultaCard.tsx
│   └── index.ts
├── Props tipadas localmente
├── Callbacks para comunicação
└── App.tsx simplificado (~120 linhas)
```

---

## Conclusão

Hoje demos um passo GIGANTESCO em direção a uma arquitetura profissional e escalável!

**O Que Aprendemos:**

- **Componentização** - Extrair código em partes reutilizáveis
- **Props Tipadas** - Interface do componente com TypeScript
- **Callbacks** - Comunicação filho → pai mantendo fluxo unidirecional
- **Encapsulamento** - JSX + lógica + estilos juntos
- **Organização** - Types locais vs globais
- **Redução de Complexidade** - 200 linhas → 120 linhas

---

## Próxima Aula

Vamos trabalhar com múltiplas consultas usando arrays e renderização dinâmica com `.map()`. Também entenderemos quando usar `FlatList` para performance em listas longas!

---

> Por hoje é isso! 🚀
