# ♿ CitizenTech · Acessibilidade Urbana ♿

## 🌐 Global Solution 2026.2 · FIAP 🌐
## 💻 Engenharia de Software · IA & Machine Learning 💻

### 👥 Integrantes
| Nome | RM |
|---|---|
| André Luiz Fernandes de Queiroz | 554503 |
| Marcos Vinicius da Silva Costa | 555490 |
| Paulo Poças | 556080 |
| Rafael Bocchi | 557603 |
| Rafael Federici de Oliveira | 554736 |

## 📋 Ficha da CitizenTech
| Campo | Conteúdo |
|---|---|
| Cidadão beneficiado | Pessoas com deficiência ou mobilidade reduzida |
| Problema em uma frase | Estabelecimentos têm barreiras de acessibilidade e não sabem quais são nem a quem recorrer |
| Como funciona | Questionário e medições guiadas → **triagem** de possíveis barreiras → indicação de parceiros especializados |
| O que **não** fazemos | Obras, adaptações ou cursos; o diagnóstico é triagem e **não substitui** avaliação técnica presencial |
| Papel do ML | Estimar a **probabilidade de faltar rampa** em um estabelecimento, para priorizar quem deve ser encaminhado primeiro a um parceiro |
| ODS | 4 (educação inclusiva), 10 (redução das desigualdades) e 11 (cidades acessíveis) |

## 📦 O que esta entrega contém
Ciclo completo de **Machine Learning** sobre dados **reais e públicos** do **Censo Escolar (INEP)**: preparo dos dados, treino de um **XGBoost** com 12 combinações de hiperparâmetros, avaliação em treino e teste (inclusive **por região do Brasil**) e leitura do resultado para o cidadão.

## 🎥 Vídeo e relatório
| Item | Link |
|---|---|
| Vídeo |  |
| Relatório | [`RELATORIO_ML.md`](ml/RELATORIO_ML.md) |
| Notebook | [`GS_CitizenTech_IAML.ipynb`](GS_CitizenTech_IAML.ipynb) |

🤖 **Pergunta de ML:** dadas as características de uma escola e os demais itens de acessibilidade que ela declara, qual a probabilidade de **faltar rampa de acesso**?

## 🏗️ Estrutura
```
.
├─ README.md
├─ requirements.txt
├─ GS_CitizenTech_IAML.ipynb
├─ dados/
│   └─ escolas_brasil_2025.csv.gz
└─ ml/
    ├─ RELATORIO_ML.md
    └─ figuras/
```

## 🎯 Como atende cada critério
| Critério (pontos) | Onde ver |
|---|---|
| **Problema e dados reais** | Notebook, seção **1**: pergunta de ML, fonte (INEP), link, período e recorte; análise por região, UF, rede e zona. |
| **Escolha justificada do modelo** | Seção **2**: comparação entre XGBoost, Random Forest, MLP, LSTM e KMeans + PCA, com o motivo da escolha |
| **Preparação e divisão treino/teste** | Seção **3**: vazamento de dados (`IN_ACESSIBILIDADE_INEXISTENTE`), código especial `88888`, seleção das 214 variáveis, divisão 80/20 **estratificada por região e alvo** e modelos de referência |
| **Treino com diferentes parâmetros** | Seção **4**: 12 combinações de 3 hiperparâmetros (`max_depth`, `learning_rate`, `n_estimators`), validação cruzada de 5 partições **somente no treino** |
| **Resultados em treino e teste** | Seção **5**: métricas lado a lado, matriz de confusão, curva ROC, desempenho por região, limiar e faixas de risco |
| **Conclusão e leitura para o cidadão** | Seção **6**: o que o resultado significa, limitações e ODS |

## 📊 Resultados em destaque
| Item | Resultado |
|---|---|
| Escolas no arquivo / em atividade | 214.192 / **180.540** |
| Escolas **sem rampa** (em atividade) | **38,0%** (62,0% com rampa) |
| Variáveis de entrada | 214 (197 de perfil e estrutura + 17 de outros itens de acessibilidade) |
| Divisão treino / teste | 144.432 / 36.108 (80/20, estratificada por região e alvo, semente 42) |
| Melhor combinação | `max_depth = 8`, `learning_rate = 0,1`, `n_estimators = 200` |
| AUC (validação cruzada, só treino) | 0,846 |

**Desempenho no teste (limiar 0,50):**
| Modelo | Acurácia | Precisão | Recall | F1 | AUC |
|---|---|---|---|---|---|
| Classe majoritária (referência) | 0,620 | 0,000 | 0,000 | 0,000 | 0,500 |
| Regressão logística (referência) | 0,767 | 0,717 | 0,638 | 0,675 | 0,822 |
| **XGBoost** | **0,787** | **0,755** | **0,650** | **0,698** | **0,850** |

- **Treino × teste (XGBoost):** AUC 0,914 × 0,850 e acurácia 0,831 × 0,787, ou seja, **sobreajuste moderado**. As combinações mais complexas chegam a AUC de treino de ~1,00 sem superar a validação da melhor combinação.
- **Matriz de confusão (teste):** 8.920 acertos ao apontar "sem rampa", 2.893 alarmes falsos, 4.811 barreiras não detectadas e 19.484 acertos em "tem rampa".
- **Faixas de risco (teste):** na faixa **Alta** (≥ 65%), 83% das escolas realmente não têm rampa; na **Média** (35–65%), 49%; na **Baixa** (< 35%), 15%.

**Por região (teste):**
| Região | % real sem rampa | AUC | Recall (corte 0,50) | Recall (corte 0,35) |
|---|---|---|---|---|
| Norte | 55,1% | 0,900 | 0,837 | 0,891 |
| Nordeste | 34,4% | 0,846 | 0,666 | 0,782 |
| Centro-Oeste | 22,4% | 0,832 | 0,398 | 0,589 |
| Sudeste | 42,6% | 0,836 | 0,641 | 0,778 |
| Sul | 27,5% | 0,800 | 0,400 | 0,586 |

O modelo **ordena bem** as escolas por risco em todas as regiões (AUC de 0,80 a 0,90), mas um corte único de 50% **deixa passar muitas barreiras** onde elas são mais raras. Por isso a plataforma deve usar **probabilidade e faixas de risco**, e não um sim/não único.

## 🗂️ Dados
| Item | Detalhe |
|---|---|
| Fonte | INEP, Censo Escolar da Educação Básica (microdados, tabela de escolas) |
| Link | https://www.gov.br/inep/pt-br/acesso-a-informacao/dados-abertos/microdados/censo-escolar |
| Período | Censo Escolar **2025** (`NU_ANO_CENSO` = 2025); arquivo original `acessibilidade_escola_2025-2026.csv` |
| Natureza | Reais, públicos e **autodeclarados**. Nenhum dado simulado e nenhum dado pessoal (removemos CNPJ e nome da escola) |
| Arquivo no repositório | `dados/escolas_brasil_2025.csv.gz`: **todas as linhas** do original, comprimidas (o original tem ~141 MB e passa do limite de 100 MB do GitHub) |

**Tratamentos:**
- Mantidas só as escolas **em atividade** (`TP_SITUACAO_FUNCIONAMENTO = 1`): 180.540 de 214.192 (29.780 paralisadas e 3.872 extintas ficam de fora).
- **Alvo:** `y_sem_rampa = 1` quando `IN_ACESSIBILIDADE_RAMPAS = 0`.
- **Vazamento removido:** `IN_ACESSIBILIDADE_INEXISTENTE = 1` implica "sem rampa" em 100% dos casos (38.790 escolas).
- Código **88888** ("não informado") em 26 colunas `QT_*` convertido em ausente.
- Removidos identificadores, datas, geografia, vínculo/parceria esparsos e colunas constantes ou com mais de 50% de ausentes. **Região e UF não entram como entrada do modelo**: servem para comparar e estratificar.

## 🔗 Integração com C# Software Development
O notebook exporta `saidas/previsoes_escolas_oof.csv` com as 180.540 escolas em atividade:

| Coluna | Significado |
|---|---|
| `co_entidade` | Código INEP da escola |
| `regiao`, `sg_uf`, `co_municipio` | Localização |
| `tp_dependencia` | 1 federal · 2 estadual · 3 municipal · 4 privada |
| `prob_sem_rampa` | Probabilidade de faltar rampa |
| `sem_rampa_declarado` | Valor real declarado (1 = sem rampa) |
| `faixa_risco` | BAIXA (101.155) · MEDIA (37.388) · ALTA (41.997) |

As probabilidades são **out-of-fold** (validação cruzada): cada escola recebe a previsão de um modelo que **não a viu** no treino, evitando resultados contaminados. A API em C# importa esse arquivo (`POST /api/risco/importar`) e, ao diagnosticar uma escola pelo código INEP, informa a probabilidade de faltar rampa e o contexto da UF.