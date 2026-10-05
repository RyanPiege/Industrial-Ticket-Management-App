# Aplicativo de Gestão de Chamados – TUR

> Uma solução integrada para priorização e gerenciamento de notas de trabalho e manutenção por turno.

![Status](https://img.shields.io/badge/status-conclu%C3%ADdo-brightgreen)
![Aviso](https://img.shields.io/badge/c%C3%B3digo-privado%20%2F%20fechado-red)
![Python](https://img.shields.io/badge/python-3.10%2B-blue)
![Google Cloud](https://img.shields.io/badge/Google%20Cloud-dados-4285F4)
![SharePoint](https://img.shields.io/badge/SharePoint-armazenagem-0078D4)
![Power Apps](https://img.shields.io/badge/Power%20Apps-interface-742774)

---

## Nota de Confidencialidade e Restrição de Código

**Atenção:** Este repositório é apenas para fins de registro e apresentação do portfólio de arquitetura do projeto. 

O sistema foi desenvolvido como uma solução corporativa interna e seu desenvolvimento encontra-se **concluído**. Devido a acordos de confidencialidade (NDA), regras de governança e proteção de propriedade intelectual, **o código-fonte, dados de teste, credenciais, scripts de ETL e arquivos de configuração não foram autorizados para exposição pública** e não estão disponíveis neste repositório.

---

## Resumo do Projeto

O **Aplicativo de Gestão de Chamados – TUR** foi concebido para centralizar, priorizar e organizar o fluxo de notas de manutenção e atendimento operacional a cada turno. A solução unificou informações que antes se encontravam dispersas em diferentes ferramentas, oferecendo à coordenação critérios automatizados, claros e objetivos para a tomada de decisão.

### Desafios Resolvidos

- **Visão Unificada:** Consolidação das notas provenientes do SAP e sistemas legados em uma única fila de visualização.
- **Priorização Automatizada:** Algoritmo de cálculo de criticidade para ordenação dinâmica das demandas no aplicativo.
- **Histórico e Rastreabilidade:** Registro completo de notas concluídas e alterações de status efetuadas pelos coordenadores.

---

## Arquitetura da Solução

```mermaid
flowchart LR
    A[Fontes de Dados<br/>SAP / Sistemas Legados] --> B[(Google Cloud<br/>Dados Brutos)]
    B --> C[Pipeline ETL em Python<br/>NumPy · Pandas · Polars]
    C --> D[(SharePoint<br/>Listas Tratadas)]
    D <--> E[Power Apps<br/>Interface e Operação]
    E --> F[Encerramento no SAP]

## Sobre o projeto

O **Aplicativo de Gestão de Chamados – TUR** organiza e prioriza as notas de manutenção de cada turno/área. Ele reúne em uma única interface informações que antes ficavam espalhadas em diferentes ferramentas e apoia a decisão da coordenação com critérios claros e objetivos.

Este repositório contém o **pipeline de dados em Python** e a documentação da integração entre **Google Cloud (GCP)**, **SharePoint** e **Power Apps**.

## Contexto e objetivos

**Desafio atual**

- As notas de manutenção chegam pelo SAP e precisam ser priorizadas a cada turno.
- A priorização depende de avaliações individuais e de ferramentas distintas.
- Não existe uma visão única da fila de notas.
- O histórico das notas concluídas é difícil de consultar.

**Objetivos**

| # | Objetivo | Descrição |
|---|----------|-----------|
| 1 | Priorização inteligente | Classificar as notas pela criticidade, direto no aplicativo Power Apps. |
| 2 | Integração de dados | Consolidar informações de diferentes ferramentas em uma única interface. |
| 3 | Decisões ágeis | Permitir decisão rápida com base em critérios claros e objetivos. |


| Camada | Responsabilidade |
|--------|------------------|
| **Google Cloud** | Armazena os dados brutos e os compartilha de forma controlada com o Python. |
| **Python** | Extrai, limpa, enriquece e pontua as notas (ETL) com NumPy e Pandas/Polars. |
| **SharePoint** | Armazena os dados tratados e faz a comunicação com o Power Apps. |
| **Power Apps** | Interface do usuário: fila priorizada, ajuste de criticidade e notas concluídas. |

As camadas são independentes: cada etapa pode evoluir sem quebrar as demais, e o Power Apps consome apenas dados já tratados.

## Fluxo do aplicativo

1. O **operador** abre a nota.
2. A **coordenação** prioriza a nota.
3. A **manutenção** executa o serviço.
4. A nota é **encerrada no SAP**.
5. O **aplicativo encerra** a nota e a move para a tela de notas concluídas.

## Critérios de priorização

As notas são exibidas por cor de acordo com o nível de criticidade:

| Nível | Cor | Significado |
|-------|-----|-------------|
| **Alta** | Vermelho | Exige atuação imediata. |
| **Média** | Amarelo | Atuação planejada dentro do turno. |
| **Baixa** | Azul | Pode aguardar janela de manutenção. |

A coordenação pode ajustar o nível de cada nota diretamente no aplicativo.

> As regras e os pesos usados no cálculo da criticidade ficam em `config/regras_criticidade.yaml`. Ajuste conforme o critério vigente da área.

## Stack tecnológica

- **Google Cloud Platform:** armazenamento e compartilhamento dos dados brutos (por exemplo Cloud Storage e BigQuery).
- **Python 3.10+:** `numpy`, `pandas` e `polars` para transformação e cálculo.
- **Microsoft SharePoint:** listas como base de dados do aplicativo.
- **Microsoft Power Apps:** aplicativo canvas conectado nativamente ao SharePoint.

## Estrutura do repositório

> Estrutura sugerida. Adapte aos nomes reais das pastas do projeto.

```text
.
├── config/
│   ├── regras_criticidade.yaml   # pesos e regras de priorização
│   └── settings.example.env      # modelo de variáveis de ambiente
├── src/
│   ├── extract/                  # leitura dos dados no GCP
│   ├── transform/                # limpeza, enriquecimento e pontuação
│   ├── load/                     # publicação no SharePoint
│   └── main.py                   # orquestra o pipeline
├── powerapps/                    # pacote exportado do aplicativo (.msapp / .zip)
├── docs/                         # apresentação, diagramas e prints
├── tests/
├── requirements.txt
└── README.md
```

O pipeline segue cinco etapas:

| Etapa | O que faz |
|-------|-----------|
| **Extrair** | Lê os dados brutos no Google Cloud. |
| **Limpar** | Remove duplicidades e padroniza campos. |
| **Enriquecer** | Cruza as notas com cadastros e informações complementares. |
| **Pontuar** | Calcula a criticidade com NumPy e Pandas/Polars. |
| **Publicar** | Grava o resultado nas listas do SharePoint. |

Recomenda-se agendar a execução de forma recorrente (por exemplo a cada início de turno) usando o agendador disponível no seu ambiente.

## SharePoint e Power Apps

- As **listas do SharePoint** guardam as notas priorizadas, as notas concluídas e os parâmetros de criticidade.
- O **Power Apps** lê e grava nessas listas pelo conector nativo do SharePoint, sem necessidade de gateway.
- O aplicativo possui duas telas principais:
  - **Chamados de Turno:** fila ordenada por criticidade, com seletor de nível por nota.
  - **Notas Concluídas:** consulta das notas já encerradas.

## Segurança e governança

- Acesso ao GCP por IAM e contas de serviço, com privilégio mínimo.
- Permissões do SharePoint e do Power Apps definidas por perfil de usuário.
- Rastreabilidade de quem priorizou cada nota e quando.
- Validação dos dados antes da publicação nas listas.

## Como contribuir

1. Crie um branch a partir do principal: `git checkout -b feature/minha-melhoria`
2. Faça commits pequenos e descritivos.
3. Abra um Pull Request explicando o que mudou e por quê.

## Contato

Equipe responsável: `<Ryan Piége; Luiz Araujo / Confiabilidade e Processos>`
Responsável técnico: `<Ryan> – <piege.dev@gmail.com>`

---

Projeto interno da **Suzano**.
