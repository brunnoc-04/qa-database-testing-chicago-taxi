# 🗄️ QA Database & Console Testing — Chicago Taxi System

## 📖 Sobre o Projeto
Projeto de Quality Assurance realizado durante o curso de QA da TripleTen (Sprint 6). 
O objetivo foi realizar testes e validações em um sistema de táxis de Chicago, 
trabalhando em duas frentes: **manipulação de logs via console Linux** e **consultas 
em banco de dados PostgreSQL** para identificar inconsistências nos dados do sistema.

## 🎯 Contexto
A equipe de desenvolvimento do aplicativo de táxis de Chicago relatou que o planejamento 
previa 10.550 veículos disponíveis, mas receberam reclamações de usuários sobre falta 
de carros. A tarefa foi investigar os dados para entender a realidade: quantos táxis 
estavam realmente disponíveis, quais empresas tinham frota insuficiente, e se havia 
erros no cálculo do coeficiente de corridas (que varia conforme o clima).

## 🛠️ Ferramentas e Tecnologias
- **Console Linux** — manipulação de logs Apache via terminal remoto
- **PostgreSQL** — consultas SQL no banco `chicago_taxi`
- **Comandos utilizados:** `grep`, `mkdir`, `cat`, `head`, `tail`, `wc`, redirecionamento `>`
- **SQL:** `SELECT`, `COUNT`, `GROUP BY`, `HAVING`, `INNER JOIN`, `CASE WHEN`, `LIKE`, `CAST`, `BETWEEN`

---

## 📋 Estrutura do Banco de Dados

O banco `chicago_taxi` contém 4 tabelas:

| Tabela | Descrição |
|--------|-----------|
| `neighborhoods` | Bairros de Chicago (ID e nome) |
| `cabs` | Táxis cadastrados (cab_id, vehicle_id, company_name) |
| `trips` | Corridas realizadas (trip_id, cab_id, datas, duração, distância, localizações) |
| `weather_records` | Registros climáticos (temperatura, descrição, timestamp) |

> ⚠️ Não há ligação direta entre `trips` e `weather_records`. A vinculação é feita 
> pelo campo de timestamp (`trips.start_ts` ↔ `weather_records.ts`).

---

## 🔧 Parte 1: Console — Manipulação de Logs

### Tarefa 1: Busca de requisições por IP
**Objetivo:** Encontrar todas as requisições vindas de IPs começando com `233.201.` 
em logs de servidor remoto (período: dezembro/2019, dia desconhecido).

**Comandos utilizados:**
```bash
cd ~/logs/2019/12
grep -R '^233.201.'
```

**Resultado:**
```
apache_2019-12-18.txt:233.201.188.154 - - [18/12/2019:21:46:01 +0000] "DELETE /events HTTP/1.1" 403 3971
apache_2019-12-21.txt:233.201.182.9 - - [21/12/2019:21:56:20 +0000] "PATCH /users HTTP/1.1" 400 4118
```

---

### Tarefa 2: Isolamento e categorização de logs de erro
**Objetivo:** Extrair logs do dia 30/12/2019 com erros 400 e 500, criar estrutura 
de diretórios e separar os erros em arquivos distintos.

**Criação da estrutura de diretórios:**
```bash
mkdir bug1
mkdir bug1/events
```

**Extração dos logs de erro do dia 30/12:**
```bash
grep '[45]00' ~/logs/2019/12/apache_2019-12-30.txt > ~/bug1/main.txt
```

**Separação por tipo de erro:**
```bash
grep '400' ~/bug1/main.txt > ~/bug1/events/400.txt
grep '500' ~/bug1/main.txt > ~/bug1/events/500.txt
```

**Resultados:**

| Arquivo | Total de linhas |
|---------|----------------|
| `400.txt` | 179 linhas |
| `500.txt` | 160 linhas |

**Amostra — 400.txt (3 primeiras linhas):**
```
80.57.170.51 - - [30/12/2019:21:35:12 +0000] "DELETE /users HTTP/1.1" 400 3623
204.235.176.118 - - [30/12/2019:21:35:13 +0000] "POST /users HTTP/1.1" 400 4704
82.95.203.67 - - [30/12/2019:21:35:19 +0000] "DELETE /lists HTTP/1.1" 400 3737
```

**Amostra — 500.txt (3 primeiras linhas):**
```
64.250.112.189 - - [30/12/2019:21:35:13 +0000] "PUT /parsers HTTP/1.1" 500 4639
193.253.101.180 - - [30/12/2019:21:35:31 +0000] "PATCH /alerts HTTP/1.1" 500 2944
197.106.117.194 - - [30/12/2019:21:35:31 +0000] "PATCH /parsers HTTP/1.1" 500 3519
```

---

## 🔍 Parte 2: Banco de Dados — Consultas SQL

### Tarefa 1: Contagem total de táxis
**Objetivo:** Verificar quantos carros estavam realmente disponíveis, já que o 
planejamento previa 10.550 veículos.

**Query:**
```sql
SELECT
    COUNT(*) as COUNT
FROM
    cabs;
```

**Resultado: 5.529 carros** (apenas 52% do planejado)

---

### Tarefa 2: Empresas com menos de 100 carros
**Objetivo:** Identificar empresas de táxi com frota insuficiente (menos de 100 carros).

**Query:**
```sql
SELECT
    company_name,
    COUNT(cab_id) AS cnt
FROM
    cabs
GROUP BY
    company_name
HAVING
    COUNT(cab_id) < 100
ORDER BY
    cnt DESC;
```

**Resultado: 51 empresas com menos de 100 carros**

| Empresa | Carros |
|---------|--------|
| Nova Taxi Affiliation Llc | 97 |
| Patriot Taxi Dba Peace Taxi Associat | 89 |
| Blue Diamond | 85 |
| Checker Taxi Affiliation | 81 |
| Chicago Medallion Management | 80 |
| Chicago Independents | 69 |
| 24 Seven Taxi | 67 |
| Checker Taxi | 60 |
| American United | 55 |
| Chicago Medallion Leasing INC | 53 |
| Top Cab Affiliation | 49 |
| KOAM Taxi Association | 48 |
| Chicago Taxicab | 38 |
| Norshore Cab | 34 |
| Gold Coast Taxi | 20 |
| Metro Group | 20 |
| Service Taxi Association | 18 |
| 5 Star Taxi | 14 |
| American United Taxi Affiliation | 8 |
| Metro Jet Taxi A | 8 |
| Setare Inc | 7 |
| Leonard Cab Co | 5 |

> *Lista completa com 51 empresas disponível no relatório original.*

---

### Tarefa 3: Classificação de condições climáticas
**Objetivo:** Criar seleção de dados para verificar o cálculo do coeficiente de 
corridas (1 para clima bom, 2 para chuva/tempestade). A equipe suspeitava de erro 
no cálculo.

**Query:**
```sql
SELECT
    ts,
    CASE
        WHEN description LIKE '%rain%' OR description LIKE '%storm%'
            THEN 'Bad'
        ELSE 'Good'
    END AS weather_conditions
FROM
    weather_records
WHERE
    ts BETWEEN '2017-11-05 00:00:00' AND '2017-11-05 23:59:59';
```

**Resultado: 24 registros (uma hora cada) classificados como Good ou Bad**

| Timestamp | Condição |
|-----------|----------|
| 2017-11-05 00:00:00 | Good |
| 2017-11-05 01:00:00 | Bad |
| 2017-11-05 02:00:00 | Good |
| 2017-11-05 03:00:00 | Good |
| 2017-11-05 04:00:00 | Bad |
| 2017-11-05 05:00:00 | Bad |
| 2017-11-05 06:00:00 | Good |
| 2017-11-05 07:00:00 | Good |
| 2017-11-05 08:00:00 | Good |
| 2017-11-05 09:00:00 | Good |
| 2017-11-05 14:00:00 | Bad |
| 2017-11-05 16:00:00 | Bad |
| 2017-11-05 18:00:00 | Bad |
| 2017-11-05 19:00:00 | Bad |
| 2017-11-05 20:00:00 | Bad |

---

### Tarefa 4: Corridas por empresa (15-16/Nov/2017)
**Objetivo:** Verificar se o número de corridas reportado pelas empresas de táxi 
corresponde aos dados do sistema após atualização de software.

**Query:**
```sql
SELECT
    cabs.company_name,
    COUNT(trips.trip_id) AS trips_amount
FROM
    cabs
INNER JOIN trips ON trips.cab_id = cabs.cab_id
WHERE
    CAST(trips.start_ts AS date) BETWEEN '2017-11-15' AND '2017-11-16'
GROUP BY
    cabs.company_name
ORDER BY
    trips_amount DESC;
```

**Resultado: 64 empresas com corridas registradas**

| Empresa | Corridas |
|---------|---------|
| Flash Cab | 19.558 |
| Taxi Affiliation Services | 11.422 |
| Medallion Leasin | 10.367 |
| Yellow Cab | 9.888 |
| Taxi Affiliation Service Yellow | 9.299 |
| Chicago Carriage Cab Corp | 9.181 |
| City Service | 8.448 |
| Sun Taxi | 7.701 |
| Star North Management LLC | 7.455 |
| Blue Ribbon Taxi Association Inc. | 5.953 |
| Choice Taxi Association | 5.015 |
| Globe Taxi | 4.383 |
| Dispatch Taxi Affiliation | 3.355 |
| Nova Taxi Affiliation Llc | 3.175 |
| Patriot Taxi Dba Peace Taxi Associat | 2.235 |
| Checker Taxi Affiliation | 2.216 |
| Blue Diamond | 2.070 |
| Chicago Medallion Management | 1.955 |
| 24 Seven Taxi | 1.775 |
| Chicago Medallion Leasing INC | 1.607 |

> *Lista completa com 64 empresas disponível no relatório original.*

---

## 📌 Aprendizados

- **Manipulação de logs no Linux:** uso de `grep` com expressões regulares para 
  filtrar requisições por padrão de IP e por código de erro HTTP
- **Estruturação de diretórios:** organização de arquivos de log por categoria 
  de erro para facilitar análise
- **SQL analítico:** aplicação de `GROUP BY` + `HAVING`, `INNER JOIN` entre 
  tabelas, `CASE WHEN` para classificação condicional, e `LIKE` para busca de padrões
- **Análise de dados:** identificação de discrepância entre planejamento (10.550 
  veículos) e realidade (5.529 veículos — apenas 52%)
- **Pensamento investigativo:** cruzamento de dados climáticos com sistema de 
  táxis para validar regra de negócio

---

## 🔗 Links
- [Relatório completo (Google Docs)](https://docs.google.com/document/d/1ypzQC1QDSJ2J1VBg5dwnVUVzzltHhOVHnQh30s3atlU/edit?tab=t.0)
