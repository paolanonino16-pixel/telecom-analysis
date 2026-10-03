# 📱 Projeto ConnectaTel — Análise de Clientes e Comportamento de Uso
Este projeto consiste em uma análise exploratória de dados (EDA), limpeza detalhada, detecção de anomalias/outliers e segmentação estratégica de clientes para a operadora fictícia de telecomunicações **ConnectaTel**.
---
## 🎯 1. Objetivo do Projeto
O objetivo principal é avaliar a base de clientes e seus hábitos de consumo de serviços (chamadas e mensagens de texto) ao longo de 2024 para responder a perguntas estratégicas de negócio:
* Identificar e tratar falhas de qualidade nos dados (valores ausentes, valores sentinels e datas fora do intervalo válido).
* Avaliar o comportamento dos clientes de acordo com o plano contratado (**Básico** vs. **Premium**).
* Analisar distribuições e investigar a presença de usuários de consumo extremo (*heavy users* / outliers).
* Criar segmentações comportamentais e etárias para orientar campanhas de migração de plano (*upsell*), retenção preventiva (*churn*) e desenvolvimento de novas ofertas.
## 📁 2. Datasets Utilizados
O projeto utiliza três bases de dados integradas via identificadores relacionais:
1. **`users.csv` (Informações cadastrais dos clientes — 4.000 registros):**
   * `user_id`: Identificador único do cliente (chave primária, de 10.000 a 13.999).
   * `first_name` e `last_name`: Nome e sobrenome do cliente.
   * `age`: Idade do usuário (incluía o sentinel `-999.0`).
   * `city`: Cidade de residência (incluía valores nulos e o sentinel `'?'`).
   * `reg_date`: Data de registro/cadastro do cliente.
   * `plan`: Tipo de plano comercial contratado (`Basico` ou `Premium`).
   * `churn_date`: Data de cancelamento do serviço (nulo se o cliente permanecer ativo).
2. **`usage.csv` (Registro detalhado de uso dos serviços — 40.000 registros em 2024):**
   * `id`: Identificador sequencial único do evento de consumo (1 a 40.000).
   * `user_id`: Identificador do usuário que utilizou o serviço (chave estrangeira).
   * `type`: Tipo de serviço utilizado (`text` para SMS ou `call` para chamada de voz).
   * `date`: Data do consumo do serviço durante o ano de 2024.
   * `duration`: Duração da chamada em minutos (nulo estrutural para mensagens).
   * `length`: Comprimento da mensagem em caracteres (nulo estrutural para chamadas).
3. **`plans` (Tabela de referência das regras comerciais — 2 linhas):**
   * Parâmetros contratuais, franquias mensais inclusas e tarifas de excedente de cada plano (`Basico` e `Premium`).
## 🛠️ 3. Etapas da Análise Realizadas
### **Etapa 1: Diagnóstico e Qualidade de Dados**
* **Detecção de Valores Ausentes:** Identificação de nulos em `city` (11,73%), `churn_date` (88,35% — clientes ativos) e `date` (0,12%).
* **Identificação de Valores Sentinels:** Detecção do sentinel numérico `-999.0` na idade (`age`) e do caractere `'?'` (96 registros) na coluna `city`.
* **Identificação de Datas Impossíveis:** Detecção de 40 clientes com data de cadastro no ano futuro de 2026 em `reg_date` (erro de captura, convertido para `NaT`).
* **Verificação de Ausência Estrutural (MAR):** Comprovação de que nulos em `duration` e `length` dependem 100% da coluna `type`.
### **Etapa 2: Limpeza e Pré-processamento**
* Imputação da idade com a **mediana (48 anos)** para as ocorrências de `-999.0`.
* Padronização de valores nulos e sentinels de cidade para a categoria `'Desconhecido'`.
* Remoção das 50 linhas ausentes de data em `usage`.
* Preservação da coluna `churn_date` para mapeamento de retenção de clientes.
### **Etapa 3: Agregação por Usuário (`user_profile`)**
* Agregação da tabela `usage` por `user_id` calculando:
  * `quant_mensagens` (total de SMS enviados no período);
  * `quant_chamadas` (total de ligações realizadas);
  * `quant_minutos_chamada` (duração total acumulada em minutos).
* União (*left merge*) com a tabela `users` criando o dataframe consolidado `user_profile`.
### **Etapa 4: Análise Exploratória e Detecção de Outliers**
* Construção de histogramas com curvas de densidade (KDE) segmentados por `plan`.
* Análise de assimetria nas variáveis de uso (distribuições assimétricas à direita com picos em consumos moderados).
* Avaliação de dispersão via **Boxplots** e cálculo de limites estatísticos pelo método **IQR (Intervalo Interquartil)**.
* Decisão justificada de **manter todos os outliers**, por representarem clientes intensivos legítimos (*heavy users*).
### **Etapa 5: Segmentação e Recomendações de Negócio**
* Segmentação de clientes por idade (`grupo_idade`): **Jovem** (< 30), **Adulto** (30 a 59) e **Idoso** (≥ 60).
* Segmentação por consumo (`grupo_uso`): **Baixo uso**, **Uso médio** e **Alto uso**.
* Formulação de recomendações estratégicas (campanhas de *upsell*, criação de plano intermediário, pacotes avulsos e retenção preventiva de churn).
## 🚀 4. Como Executar o Notebook
### Opção A: No Google Colab (Recomendada)
1. Acesse o [Google Colab](https://colab.research.google.com/).
2. Clique na aba **Upload** e selecione o arquivo do notebook (`.ipynb`).
3. No menu lateral esquerdo do Colab, clique no ícone de pasta (📁 **Arquivos**) e faça o upload dos datasets (`users.csv`, `usage.csv` e `plans.csv` se aplicável).
4. No menu superior, clique em **Ambiente de execução ➔ Executar tudo** (ou use o atalho `Ctrl + F9` / `Cmd + F9`).
### Opção B: Localmente (Jupyter Notebook / JupyterLab / VS Code)
1. Certifique-se de ter o Python 3.9+ instalado.
2. Crie e ative um ambiente virtual (opcional, mas recomendado):
   ```bash
   python -m venv venv
   source venv/bin/activate  # No Windows: venv\Scripts\activate
   ```
3. Instale as bibliotecas necessárias:
   ```bash
   pip install pandas numpy matplotlib seaborn jupyter
   ```
4. Inicie o Jupyter no terminal dentro da pasta do projeto:
   ```bash
   jupyter notebook
   ```
5. Abra o arquivo do notebook e execute as células sequencialmente.```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
# 1. Carregar os dados
users = pd.read_csv('users.csv')
usage = pd.read_csv('usage.csv')
# 2. Corrigir sentinels e datas
users['age'] = users['age'].replace(-999, np.nan)
users['age'] = users['age'].fillna(users['age'].median())
users['city'] = users['city'].replace('?', 'Desconhecido').fillna('Desconhecido')
users['reg_date'] = pd.to_datetime(users['reg_date'], errors='coerce')
users.loc[users['reg_date'].dt.year > 2024, 'reg_date'] = pd.NaT
usage = usage.dropna(subset=['date'])
# 3. Criar métricas e consolidar perfil
usage['is_text'] = (usage['type'] == 'text').astype(int)
usage['is_call'] = (usage['type'] == 'call').astype(int)
usage_agg = usage.groupby('user_id').agg(
    quant_mensagens=('is_text', 'sum'),
    quant_chamadas=('is_call', 'sum'),
    quant_minutos_chamada=('duration', 'sum')
).reset_index()
user_profile = users.merge(usage_agg, on='user_id', how='left')
# 4. Criar segmentações
def classificar_uso(r):
    if r['quant_chamadas'] < 5 and r['quant_mensagens'] < 5:
        return 'Baixo uso'
    elif r['quant_chamadas'] < 10 and r['quant_mensagens'] < 10:
        return 'Uso médio'
    return 'Alto uso'
user_profile['grupo_uso'] = user_profile.apply(classificar_uso, axis=1)
user_profile['grupo_idade'] = pd.cut(
    user_profile['age'], 
    bins=[-np.inf, 30, 60, np.inf], 
    right=False, 
    labels=['Jovem', 'Adulto', 'Idoso']
)
```
## 👥 Autor
Projeto desenvolvido como parte do programa de formação em Análise de Dados (Sprint 8: Análises Estatísticas e Qualidade de Dados).
