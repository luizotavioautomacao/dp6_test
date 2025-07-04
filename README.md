# Proposta de Solução - Teste Técnico DP6 - Engenharia de Dados

## 🔢 Problema Escolhido

**Monitoramento de campanhas Facebook e Google com dados do Google Analytics 4 (GA4), com dashboards diários.**

> ⚠️ **Atenção**
> Soluções baseadas em *event-driven pipelines* (como triggers em tempo real) costumam gerar custos elevados e complexidade operacional que raramente se justificam no contexto de marketing.
> Um **dashboard diário** já atende muito bem às necessidades analíticas.
> Para isso, o ideal é ter uma aplicação que consulte diretamente os dados disponíveis nos bancos de dados das aplicações.

---

## 🧭 Fluxo Completo de Dados (Fonte: Aplicações Web/App e Sistemas Internos)

```text
+-------------------------+     +----------------------------------+
| Aplicações Web (ecom)   |     | Aplicativos Mobile (iOS/Android) |
+-----------+-------------+     +-------------+--------------------+
            |                               |
            |                               |
            v                               v
 +---------------------+        +-------------------------+
 | GA4 (via gtag.js /  |        | Firebase + GA4 SDK      |
 | Google Tag Manager) |        +-------------------------+
 +----------+----------+                   |
            |                              v
            |                +-----------------------------+
            |                | Google Analytics (GA4)      |
            +--------------->| Eventos e conversões        |
                             +-------------+---------------+
                                           |
+------------------------------+     +------------------------+
| Sistemas Internos (ERP/CRM) |<---> |  Python ETL Scripts    |
+------------------------------+     |  (EC2 / Lambda / cron) |
                                     +-----------+------------+
                                                 |
                        +----------------------------+
                        | Snowflake (RAW + STG layers)|
                        +----------------------------+
                                   |
                            +------+------+
                            |   dbt models |
                            +------+------+
                                   |
                             +-----v------+
                             | Final tables|
                             | (CPA, ROAS) |
                             +-----+-------+
                                   |
                    +--------------v-------------------------+
                    |  Dashboard (BI / Streamlit/ Metabase)  |
                    +-----------------------------------------+
```

Este fluxo representa a origem dos dados diretamente das aplicações (frontend e backend) e sistemas internos, além das integrações com plataformas externas de mídia e analytics.

---

## 📊 A. Arquitetura da Solução (Base: Snowflake)

```text
                  +------------------------+
                  | Google Analytics (GA4) |
                  +-----------+------------+
                              |
                              v
    +------------------------------+     +------------------------+
    | Facebook Ads API (Meta)      |<--->|  Python ETL Scripts    |
    +------------------------------+     |  (EC2 / Lambda / cron) |
                              |             +-----------+------------+
                              v                         |
                       +----------------------------+   |
                       | Snowflake (RAW + STG layers)|<--+
                       +----------------------------+
                                  |
                           +------+------+
                           |   dbt models |
                           +------+------+
                                  |
                            +-----v------+
                            | Final tables|
                            | (CPA, ROAS) |
                            +-----+-------+
                                  |
              +-------------------▼---------------------+
              |  Dashboard (BI / Streamlit / Metabase)  |
              +-----------------------------------------+
```

---

## 📦 B.1 Componentes e Descrição

| Componente                 | Função                                           | Link de Referência                                                                                                                                     |
| -------------------------- | ------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| GA4                        | Coleta de eventos e conversões de Web/App        | [https://support.google.com/analytics/answer/10089681](https://support.google.com/analytics/answer/10089681)                                           |
| Google Tag (gtag.js) / GTM | Coleta de dados diretamente do site              | [https://developers.google.com/tag-platform/gtagjs](https://developers.google.com/tag-platform/gtagjs)                                                 |
| Facebook Ads API           | Métricas de campanhas de mídia                   | [https://developers.facebook.com/docs/marketing-api/](https://developers.facebook.com/docs/marketing-api/)                                             |
| Python Scripts             | Extração e envio de dados para Snowflake         | [https://docs.python.org/3/](https://docs.python.org/3/)                                                                                               |
| Snowflake                  | Data warehouse em nuvem para dados estruturados  | [https://docs.snowflake.com/](https://docs.snowflake.com/)                                                                                             |
| dbt                        | Modelagem e transformação de dados               | [https://docs.getdbt.com/](https://docs.getdbt.com/)                                                                                                   |
| Streamlit (Dashboards)     | Visualização interativa e criação de Data Apps   | [https://streamlit.io/](https://streamlit.io/)                                                                                                         |
| Metabase                   | Dashboard exploratório para negócios e marketing | [https://www.metabase.com/docs/latest/](https://www.metabase.com/docs/latest/)                                                                         |
| Power BI (Dashboards)      | Visualização de dados e dashboards corporativos  | [https://powerbi.microsoft.com/](https://powerbi.microsoft.com/)                                                                                       |
| Airflow (opcional)         | Orquestração de pipelines, monitoramento         | [https://airflow.apache.org/](https://airflow.apache.org/)                                                                                             |
| Roles/Masking (Snowflake)  | Segurança e controle de acesso a dados sensíveis | [https://docs.snowflake.com/en/user-guide/security-access-control-overview](https://docs.snowflake.com/en/user-guide/security-access-control-overview) |


## B.2 Melhorias 

| Componente              | Melhoria Proposta                         | Impacto Esperado                     |
|-----------------------  |-------------------------------------------|--------------------------------------|
| **GA4 Integration**     | Hotspots com tutorial contextual          | ↓ 30% tempo onboarding               |
| **Snowflake STG Layer** | Modelo RACI incorporado                   | ↑ Clareza responsabilidades          |
| **dbt Models**          | Validação contra ref. architecture (GCP)  | ↑ Alinhamento boas práticas          |
| **Dashboard Streamlit** | Progresso trial + gamification            | ↑ 25% conversão trial>paid           |

---

## 📄 C. Defesa da Solução

* **Escalabilidade:** Snowflake permite computação elástica para grandes volumes com performance.
* **Modularidade:** dbt permite reuso, CI/CD, testes automáticos e documentação integrada.
* **Segurança:** Uso de roles, masking e RLS garante conformidade com LGPD.
* **Governança de dados:** Auditoria com logs de execução, testes de qualidade e views controladas.
* **Custo-benefício:** Ferramentas open-source e cloud-friendly, com pagamento sob demanda.
* **Integração:** Conexões fáceis com ferramentas de BI, CRM, mídia e APIs.
* **Rastreabilidade completa:** Pipeline controlado do front-end (via gtag.js) até os dashboards finais.

---

## 📊 D. Premissas do Cliente

* Acesso às credenciais das APIs (GA4, Meta Ads, Google Ads).
* Inclusão do script `gtag.js` ou uso do Google Tag Manager no site e app.
* Permissão para leitura/escrita no Snowflake (warehouse + database + schemas).
* Contas e roles criadas: `etl_engineer`, `marketing_analyst`, `external_agency`.
* Equipe de marketing define métricas (ex: o que é "conversão").
* Acesso à ferramenta de visualização (BI, Streamlit, etc).

---

## 📊 E. Boas Práticas / Pontos de Atenção

### 🔢 dbt

* Nomear modelos com prefixo: `stg_`, `int_`, `fct_`, `dim_`
* Usar `dbt tests` (`not_null`, `unique`, etc.) para qualidade dos dados
* Criar `schema.yml` com descrição e documentação das métricas

### 🔐 Segurança

* Criar roles distintas com acesso limitado por camada (RAW, STG, MARTS)
* Aplicar `Dynamic Data Masking` para esconder PII
* Usar `Row-Level Security` em views por agência ou time

### ⏱️ Monitoramento

* Logs de execução via dbt Cloud e/ou Airflow
* Tabela `data_audit_log` para controle de execuções
* Alertas por e-mail ou Slack em caso de falha

---

## ⌛ Tempo de Implementação Estimado para primeira versão da Solução (v1)

Este prazo contempla a **primeira release funcional (MVP)** da solução, entregando valor inicial com dashboards e métricas-chave para o time de marketing. Fases adicionais — como automações, exportações avançadas, views segmentadas, observabilidade, catalogação e integração com CRM — podem ser previstas em ciclos futuros de entrega.

### 📦 Sugerido: Estratégia de Releases Incrementais (DataOps Inspired)

* **v1.0 – MVP Funcional:** coleta, modelagem básica e dashboard com CPA/ROAS
* **v1.1 – Observabilidade:** integração de testes com dbt + alertas automatizados
* **v1.2 – Governança e Metadados:** catálogo de dados com DataHub + controle de acesso
* **v2.0 – Enriquecimento + CRM:** integração com dados internos (ERP/CRM), uso de Data Contracts e possíveis exports para parceiros

| Etapa                          | Prazo/Dias  | Descrição                                                      |
| ------------------------------ | ----------- | -------------------------------------------------------------- |
| Levantamento de Requisitos     | 1           | Entendimento com stakeholders, definição de escopo e métricas  |
| Planejamento e Setup           | 2           | Planejamento detalhado, setup de Snowflake, GA4, GTM           |
| Ingestão via API + GA4         | 8           | Implementação dos scripts Python, coleta GA4, testes unitários |
| Modelagem dbt (STG + MARTS)    | 6           | Criação de modelos, testes dbt, documentação                   |
| Dashboards (Streamlit/PowerBI) | 7           | Criação de dashboards com filtros, CPA, ROAS, etc              |
| Monitoramento/alertas          | 3           | Setup de alertas, logs, tabela de auditoria                    |
| **Total estimado (v1)**        | **28 dias** | Primeira entrega com valor tangível para o cliente             |


## 📊 Matriz RACI (Baseado em PMBOK)

| Atividade                  | Eng. Dados | Anal. BI | Product Owner | Marketing |
|----------------------------|------------|----------|---------------|-----------|
| Coleta dados GA4           | R          | C        | I             | A         |
| Modelagem métricas ROAS    | A          | R        | C             | I         |
| Atualização dashboards     | I          | R        | C             | A         |
| Validação KPI campanhas    | C          | A        | R             | I         |

**Legenda:**  
- **R** (Responsible): Executor direto da tarefa  
- **A** (Accountable): Última autoridade/responsável final  
- **C** (Consulted): Deve ser consultado  
- **I** (Informed): Deve ser informado  

## 🚀 Customer Journey Map 2.0 (Foco KPIs)

| Fase            | KPI Alvo                | Melhoria Proposta                  |
|-----------------|-------------------------|------------------------------------|
| Conscientização | Taxa clique->lead       | Landing pages segmentadas          |
| Consideração    | Tempo avaliação         | Templates pré-configurados         |
| Decisão         | Conversão trial         | Progress bar + lembretes           |

**Insights das referências:**
1. Adoção de "double pipeline" para opções (OHL)
2. Padronização certificações para alunos
3. Hotspots com tutoriais contextuais

---

## 🔄 Ciclo de Melhoria Contínua

1. **Coleta**:
   - Implementar data audit log (nível linha)
   - Rastrear MOTs (Momentos de Verdade) críticos

2. **Processamento**:
   - Validação contra [GCP Reference Architecture](https://cloud.google.com/architecture/application-development)
   - Testes dbt com amostras reais

3. **Visualização**:
   Data App em Streamilit para interações com os dados
---

## 🔺 Alinhamento com Requisitos DP6

| Requisito                                    | Atendido?   | Observação                                |
| -------------------------------------------- | ----------  | ----------------------------------------- |
| Coleta Web/App + Cross-device tracking       | ✅ Sim      | GA4 com enrich de CRM por user\_id        |
| Processamento em larga escala + Orquestração | ✅ Sim      | Snowflake + dbt + Airflow (opcional)      |
| Qualidade do dado + ciclo de vida            | ✅ Sim      | dbt tests + data\_audit\_log              |
| Segurança e acessos                          | ✅ Sim      | Roles, masking e RLS no Snowflake         |
| Integração com BI, CRM, plataformas          | ✅ Sim      | Looker Studio, Streamlit, exports via API |
| Envio de dados em tempo real                 | ⚠️ Parcial  | Possível com Snowpipe/Kafka (alto custo)  |

---

**Autor:** Luiz Otávio Mendes de Oliveira
**Cartão de vistita:** [https://luizotavio.netlify.app](https://luizotavio.netlify.app)
