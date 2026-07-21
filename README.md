# 🏥 Previsão de Absenteísmo em Consultas Médicas (Medical No-Show)

Este repositório documenta o pipeline completo de Ciência de Dados — desde o pré-processamento, engenharia de recursos e análise exploratória até a modelagem preditiva utilizando **Random Forest** com **Validação Cruzada Aninhada (Nested CV)** — focado na previsão de ausência de pacientes em consultas agendadas (*No-Show*).

---

## 📌 1. Sobre o Dataset

* **Fonte original:** [Kaggle - Medical Appointment No Shows](https://www.kaggle.com/datasets/joniarroba/noshowappointments)
* **Tamanho do Dataset:** ~110.527 registros e 14 variáveis originais.
* **Variável Alvo:** `No-show` (`0` = Compareceu / `1` = Faltou).

### Dicionário de Variáveis Originais

| Variável | Descrição |
| --- | --- |
| `PatientId` | Identificador único do paciente |
| `AppointmentID` | Identificador único da consulta |
| `Gender` | Gênero do paciente (M/F) |
| `ScheduledDay` | Data/hora em que a consulta foi agendada |
| `AppointmentDay` | Data marcada para a consulta |
| `Age` | Idade do paciente |
| `Neighbourhood` | Bairro onde se localiza o consultório/hospital |
| `Scholarship` | Indica se o paciente recebe o benefício Bolsa Família (0/1) |
| `Hipertension` | Indica se o paciente é hipertenso (0/1) |
| `Diabetes` | Indica se o paciente é diabético (0/1) |
| `Alcoholism` | Indica se o paciente possui dependência alcoólica (0/1) |
| `Handcap` | Indica deficiência física (graus de 0 a 4) |
| `SMS_received` | Quantidade/Confirmação de SMS enviados ao paciente (0/1) |
| `No-show` | Variável Alvo (`No` = compareceu, `Yes` = não compareceu) |

---

## ⚙️ 2. Pipeline de Pré-processamento e Engenharia de Recursos

1. **1. Remoção de Identificadores:** Limpeza Estrutural.
Remoção direta das colunas PatientId, AppointmentID e Neighbourhood por não agregarem capacidade de generalização espacial/individual direta ao modelo.


2. **2. Recodificação de Atributos:** Mapeamento Categórico.
Mapeamento de Gender ($M=0, F=1$) e da classe alvo No-show ($No=0, Yes=1$).


3. **3. Extração Temporal:** Feature Engineering.
Extração do dia da semana, dia do mês, mês e flag de fim de semana a partir de AppointmentDay, além da categorização em turnos (matutino, vespertino, noturno) via ScheduledDay com aplicação de One-Hot Encoding.


4. **4. Qualidade e Ausentes:** Tratamento de Dados.
Remoção de registros inconsistentes (idades negativas) e preenchimento de eventuais dados nulos via mediana.


---

## 🔍 3. Análise Exploratória dos Dados (EDA)

A etapa de análise exploratória concentrou-se nos seguintes pontos:

1. **Distribuição da Variável Alvo:** Avaliação do desbalanceamento de classes entre comparecimentos e faltas.
2. **Perfil Demográfico:** Análise da distribuição de idade em relação ao absenteísmo.
3. **Análise de Correlação:** Mapeamento da matriz de correlação entre variáveis comportamentais (como envio de SMS, comorbidades e perfil socioeconômico) e a variável resposta.

---

## 🤖 4. Modelagem e Estratégia de Validação

Para obter uma estimativa realista do desempenho do modelo sem vazamento de dados (*data leakage*), foi empregada a técnica de **Validação Cruzada Aninhada (Nested CV 5x5)**:

* **Loop Interno (Inner CV - 5 dobras):** Otimização de hiperparâmetros com `GridSearchCV`, utilizando a métrica **AUC-ROC** para seleção do melhor conjunto de parâmetros do **Random Forest**.
* **Loop Externo (Outer CV - 5 dobras):** Avaliação imparcial da capacidade de generalização do modelo em 5 partições de teste independentes.

### Grade de Hiperparâmetros Avaliada

* **Número de Árvores (`n_estimators`):** `[100, 200, 500]`
* **Profundidade Máxima (`max_depth`):** `[3, 5, 7]`
* **Amostras Mínimas para Divisão (`min_samples_split`):** `[2, 5]`
* **Peso das Classes (`class_weight`):** `'balanced'`

---

## 📊 5. Resultados do Experimento

Métricas médias obtidas através das 5 dobras externas de teste da Validação Cruzada Aninhada:

| Métrica | Média | Desvio Padrão |
| --- | --- | --- |
| **AUC ROC** | **0.6321** | 0.0031 |
| **Sensibilidade (Recall)** | **0.5444** | 0.0104 |
| **Especificidade** | **0.6477** | 0.0088 |
| **Acurácia** | **0.6269** | 0.0049 |
| **G-Mean** | **0.5937** | 0.0017 |

> **Conclusão:** O modelo demonstrou alta consistência e baixo desvio padrão entre as dobras. A estratégia de ponderação de classes (`balanced`) permitiu equilibrar o ganho em **Sensibilidade (54.44%)** e **Especificidade (64.77%)**, resultando em uma média geométrica (**G-Mean**) de **0.5937**.
