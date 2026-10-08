# Supply Chain ML — Previsão de Atraso de Fornecedores

Projeto de Machine Learning aplicado a Supply Chain que prevê, **no momento da criação do pedido de compra**, a probabilidade de um fornecedor entregar com atraso.

> **Status:** em desenvolvimento (projeto de estudo). Veja o [roadmap](#roadmap) para o que já foi concluído.

---

## Sumário

- [Problema de negócio](#problema-de-negócio)
- [Objetivo](#objetivo)
- [Dados](#dados)
- [Cuidado com Data Leakage](#cuidado-com-data-leakage)
- [Abordagem](#abordagem)
- [Estrutura do projeto](#estrutura-do-projeto)
- [Como executar](#como-executar)
- [Roadmap](#roadmap)
- [Limitações](#limitações)
- [Tecnologias](#tecnologias)
- [Autor](#autor)

---

## Problema de negócio

Atrasos de fornecedores causam ruptura de estoque, fretes emergenciais, vendas perdidas e desgaste nas relações comerciais. Hoje, o comprador costuma descobrir o atraso **depois** que ele acontece.

Este projeto transforma essa gestão **reativa** em **preventiva**: o modelo estima o risco de cada pedido e permite priorizar o acompanhamento antes que o problema ocorra.

```text
Problema de negócio → Problema de ML → Dados → Modelo → Previsão → Decisão
```

## Objetivo

Construir um modelo de **classificação supervisionada binária** que receba as características de um novo pedido e retorne algo como:

```text
Pedido: PED01872
Fornecedor: Fornecedor X
Probabilidade de atraso: 82%
Classificação: CRÍTICO
```

**Variável-alvo:** `Atrasou` (0 = não atrasou, 1 = atrasou).

### Níveis de risco (regra inicial)

| Probabilidade | Nível | Ação sugerida |
|---|---|---|
| 0% – 30% | BAIXO | Acompanhamento normal |
| 31% – 60% | MÉDIO | Monitorar |
| 61% – 80% | ALTO | Acompanhar previsão de entrega |
| 81% – 100% | CRÍTICO | Contato imediato com o fornecedor |

> Esses limites são um ponto de partida. Em um cenário real, devem ser definidos a partir dos custos e objetivos da empresa.

## Dados

Arquivo: `data/raw/projeto_previsao_atraso_fornecedor.xlsx`

> **Todos os dados são fictícios (sintéticos)**, criados exclusivamente para estudo e portfólio.

| Aba | Conteúdo |
|---|---|
| `LEIA-ME` | Documentação da base |
| `PEDIDOS_ML` | Tabela principal: 1.500 pedidos × 30 colunas (1 linha = 1 pedido) |
| `RESULTADO_POS_ENTREGA` | Resultado real após a entrega (**não** usar como entrada) |
| `FORNECEDORES` | Cadastro de 20 fornecedores |
| `PRODUTOS` | Cadastro de 60 produtos |
| `DICIONARIO_DADOS` | Descrição de cada campo |

**Resumo da base principal (`PEDIDOS_ML`):**

- Período: 01/01/2025 a 30/09/2026
- 1 coluna de data, 14 numéricas, 14 categóricas e a variável-alvo
- Sem valores ausentes e sem duplicatas
- Classes desbalanceadas: **65% sem atraso** e **35% com atraso**
- O alvo possui tolerância de 1 dia (pedidos entregues com 1 dia de diferença são classificados como "no prazo")

## Cuidado com Data Leakage

Algumas informações só existem **depois** da entrega e revelam a resposta que queremos prever. Elas **não** entram como variáveis de entrada:

```text
Data_Entrega_Real
Lead_Time_Real_Dias
Dias_Atraso
Status_Entrega
```

Apenas informações disponíveis **no momento do pedido** são utilizadas. As features criadas também respeitam essa regra (nenhuma usa informação futura).

## Abordagem

1. Entendimento do problema e exploração inicial dos dados
2. Análise exploratória (EDA) e verificação de Data Leakage
3. Feature Engineering (mês, dia da semana, distância por peso, cobertura de estoque, etc.)
4. Tratamento de variáveis categóricas (One-Hot Encoding)
5. Divisão **temporal** treino/teste (dados antigos para treino, recentes para teste)
6. Modelo baseline (`DummyClassifier`)
7. Comparação de modelos: Logistic Regression, Decision Tree, Random Forest e Gradient Boosting/XGBoost
8. Avaliação: Accuracy, Precision, Recall, F1-Score, ROC-AUC e matriz de confusão (com atenção especial ao **Recall**)
9. Ajuste de hiperparâmetros (`GridSearchCV` / `RandomizedSearchCV`)
10. Probabilidade de atraso (`predict_proba`) e níveis de risco
11. Interpretabilidade (feature importance, coeficientes, SHAP)
12. Entrega: API (FastAPI), interface (Streamlit) e dashboard (Power BI)

## Estrutura do projeto

```text
previsao-atraso-fornecedor/
│
├── data/
│   ├── raw/                  # Dados originais (nunca editar)
│   └── processed/            # Dados tratados e prontos para modelagem
│
├── notebooks/                # Exploração e experimentos
│
├── src/
│   ├── data_processing.py    # Carga e limpeza dos dados
│   ├── feature_engineering.py# Criação de novas variáveis
│   ├── train.py              # Treinamento dos modelos
│   ├── predict.py            # Previsão para novos pedidos
│   └── evaluation.py         # Métricas e gráficos de avaliação
│
├── models/                   # Modelos treinados (.joblib)
├── reports/                  # Gráficos e relatórios
├── dashboard/                # Arquivos do Power BI
├── requirements.txt
└── README.md
```

## Como executar

**1. Clonar o repositório**

```bash
git clone https://github.com/SEU_USUARIO/previsao-atraso-fornecedor.git
cd previsao-atraso-fornecedor
```

**2. Criar e ativar um ambiente virtual**

```bash
python -m venv .venv

# Windows
.venv\Scripts\activate

# Linux / macOS
source .venv/bin/activate
```

**3. Instalar as dependências**

```bash
pip install -r requirements.txt
```

**4. Abrir o notebook de exploração**

```bash
jupyter notebook notebooks/
```

> As instruções de treino, API e interface serão adicionadas conforme o projeto avança.

## Roadmap

- [x] Etapa 1 — Entendimento do problema e exploração inicial da base
- [ ] Etapa 2 — Análise exploratória e Data Leakage
- [ ] Etapa 3 — Feature Engineering e tratamento de categóricas
- [ ] Etapa 4 — Divisão temporal e modelo baseline
- [ ] Etapa 5 — Treinamento e comparação de modelos
- [ ] Etapa 6 — Avaliação, matriz de confusão e escolha do modelo
- [ ] Etapa 7 — Ajuste de hiperparâmetros
- [ ] Etapa 8 — Probabilidade, níveis de risco e função de previsão
- [ ] Etapa 9 — Interpretabilidade (SHAP)
- [ ] Etapa 10 — Salvar modelo, API (FastAPI) e interface (Streamlit)
- [ ] Etapa 11 — Dashboard Power BI e consultas SQL
- [ ] Etapa 12 — Testes completos, análise de negócio e documentação final

## Limitações

- Os dados são **fictícios** e não representam perfeitamente uma operação real.
- Na base, as métricas históricas de cada fornecedor parecem constantes ao longo do tempo; em uma empresa real elas variam a cada data.
- Previsão **não é certeza**: o modelo estima probabilidades.
- Fornecedores mudam de comportamento e os padrões mudam; o modelo precisa ser **monitorado e reavaliado** periodicamente.
- A qualidade dos dados influencia diretamente a qualidade do modelo.
- O modelo **apoia**, e não substitui, a decisão humana do comprador.

## Tecnologias

Python · pandas · NumPy · matplotlib · seaborn · scikit-learn · XGBoost · SHAP · joblib · FastAPI · Streamlit · Power BI · SQL (PostgreSQL)

## Evoluções futuras

Previsão de dias de atraso, previsão de demanda, previsão de ruptura, score de fornecedores, otimização de compras, estoque de segurança, detecção de anomalias e recomendação de transportadora, formando uma plataforma de **Supply Chain Intelligence**.

## Autor

**Victor R Barbaresco**
Estudante de Ciência da Computação
[LinkedIn](https://www.linkedin.com/in/victor-rigonati-barbaresco/) · [GitHub](https://github.com/rigonat1)