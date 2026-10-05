# Aplicativo de Gestão de Chamados – TUR

> Uma solução integrada para transformar a maneira como priorizamos e gerenciamos notas de trabalho.

![Status](https://img.shields.io/badge/status-em%20desenvolvimento-yellow)
![Python](https://img.shields.io/badge/python-3.10%2B-blue)
![Google Cloud](https://img.shields.io/badge/Google%20Cloud-dados-4285F4)
![SharePoint](https://img.shields.io/badge/SharePoint-armazenagem-0078D4)
![Power Apps](https://img.shields.io/badge/Power%20Apps-interface-742774)

---

## Índice

- [Sobre o projeto](#sobre-o-projeto)
- [Contexto e objetivos](#contexto-e-objetivos)
- [Arquitetura](#arquitetura)
- [Fluxo do aplicativo](#fluxo-do-aplicativo)
- [Critérios de priorização](#critérios-de-priorização)
- [Stack tecnológica](#stack-tecnológica)
- [Estrutura do repositório](#estrutura-do-repositório)
- [Como começar](#como-começar)
- [Configuração](#configuração)
- [Executando o pipeline](#executando-o-pipeline)
- [SharePoint e Power Apps](#sharepoint-e-power-apps)
- [Segurança e governança](#segurança-e-governança)
- [Roadmap](#roadmap)
- [Como contribuir](#como-contribuir)
- [Contato](#contato)

---

## Sobre o projeto

O **Aplicativo de Gestão de Chamados – TUR** organiza e prioriza as notas de manutenção de cada turno. Ele reúne em uma única interface informações que antes ficavam espalhadas em diferentes ferramentas e apoia a decisão da coordenação com critérios claros e objetivos.

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

## Arquitetura

```mermaid
flowchart LR
    A[Fontes<br/>SAP e demais sistemas] --> B[(Google Cloud<br/>dados brutos)]
    B --> C[Python<br/>NumPy · Pandas · Polars<br/>ETL]
    C --> D[(SharePoint<br/>listas de dados tratados)]
    D <--> E[Power Apps<br/>interface nativa]
    E --> F[Conclusão no SAP]
```

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

## Como começar

### Pré-requisitos

- Python 3.10 ou superior
- Acesso ao projeto no Google Cloud, com conta de serviço autorizada
- Acesso ao site do SharePoint onde ficam as listas
- Licença do Power Apps para editar e publicar o aplicativo

### Instalação

```bash
# 1. Clonar o repositório
git clone <URL-DO-REPOSITORIO>
cd <NOME-DO-REPOSITORIO>

# 2. Criar e ativar o ambiente virtual
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate

# 3. Instalar as dependências
pip install -r requirements.txt
```

## Configuração

Copie o modelo de variáveis de ambiente e preencha com os valores do seu ambiente:

```bash
cp config/settings.example.env .env
```

| Variável | Descrição |
|----------|-----------|
| `GCP_PROJECT_ID` | Identificador do projeto no Google Cloud. |
| `GCP_BUCKET` / `GCP_DATASET` | Local dos dados brutos. |
| `GOOGLE_APPLICATION_CREDENTIALS` | Caminho do arquivo de credencial da conta de serviço. |
| `SHAREPOINT_SITE_URL` | URL do site do SharePoint. |
| `SHAREPOINT_LIST_NOTAS` | Nome da lista com as notas priorizadas. |
| `SHAREPOINT_LIST_CONCLUIDAS` | Nome da lista com as notas concluídas. |

> **Atenção:** nunca faça commit de credenciais, chaves ou do arquivo `.env`. Mantenha esses itens no `.gitignore`.

## Executando o pipeline

```bash
python -m src.main
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

Para importar o aplicativo, abra o Power Apps, escolha **Aplicativos → Importar pacote de tela de tela** e selecione o arquivo da pasta `powerapps/`. Em seguida, reconecte a fonte de dados às listas do seu site.

## Segurança e governança

- Acesso ao GCP por IAM e contas de serviço, com privilégio mínimo.
- Permissões do SharePoint e do Power Apps definidas por perfil de usuário.
- Rastreabilidade de quem priorizou cada nota e quando.
- Validação dos dados antes da publicação nas listas.

## Roadmap

- [ ] Consolidar as cargas do GCP e os pipelines Python
- [ ] Validar a priorização com a coordenação
- [ ] Expandir o aplicativo para outras áreas
- [ ] Incluir indicadores de atendimento
- [ ] Usar o histórico de notas para refinar as regras de priorização

## Como contribuir

1. Crie um branch a partir do principal: `git checkout -b feature/minha-melhoria`
2. Faça commits pequenos e descritivos.
3. Abra um Pull Request explicando o que mudou e por quê.

## Contato

Equipe responsável: `<NOME DA EQUIPE / ÁREA>`
Responsável técnico: `<NOME> – <E-MAIL>`

---

Projeto interno da **Suzano**. Defina aqui a licença ou a política de uso do repositório.
