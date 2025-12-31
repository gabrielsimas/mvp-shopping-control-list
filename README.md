## Livros que me apoiaram no aprendizado deste MVP
|Serviço	| Finalidade	|Tipo	|Nome	|Autores	|Publicação|	Tamanho|
| ---- | ---- | ---- | ---- | ---- | ---- | ---- |
|Apache Airflow|	Automação de Pipeline|	Livro|	Data Pipelines with Apache Airflow|	Julian de Ruiter, Bas Harenslak|	mai/21|	480|
|Apache Airflow|	Automação de Pipeline|	Livro|	Apache Airflow Best Practices|	Dylan Intorf, Dylan Storey, Kendrick van Doorn|	out/24|	188|
|Apache Iceberg|	Open Table Format|	Livro|	Apache Iceberg: The Definitive Guide|	Tomer Shiran, Jason Hughes, Alex Merced|	mai/24|	344|
|dbt|	Transformação de Dados|	Livro|	Unlocking dbt: Design and Deploy Transformations in Your Cloud Data Warehouse|	Dustin Dorsey, Cameron Cyr|	set/25|	491|
dbt|	Transformação de Dados|	Livro|	Data Engineering with dbt|	Roberto Zagni|	jun/23|	578|
|dbt	|Transformação de Dados|	Livro|	Analytics Engineering with SQL and dbt|	Rui Pedro Machado, Helder Russa|	dez/23|	324|
|DuckDb	|Transformação de Dados|	Livro|	Getting Started with DuckDB|	Simon Aubury, Ned Letcher|	jun/24|	382|
|DuckDb|	Transformação de Dados|	Livro	|DuckDB: Up and Running|	Wei-Meng Lee|	dez/24	|308|
|DuckDb|	Transformação de Dados|	Livro|	DuckDB in Action	|Michael Simons, Mark Needham, Michael Hunger|	ago/24	|312|

## **1) Apache Airflow**
Os dois livros escolhidos para o estudo do *Airflow* se complementam perfeitamente para o estudo e criação do pipeline:

- ### **Data Pipelines with Apache Airflow (Julian de Ruiter & Bas Harenslak)**:

    - **Onde ele entra no MVP**: Este é o "guia definitivo" para entender a anatomia das DAGs. Ele vai ajudar a estruturar a classe `IngestionManager` de forma que se torne resiliente.

    - **Aplicação Prática**: Usaremos para dominar o conceito de Idempotência. No nosso MVP, se a ingestão da planilha de compras falhar no meio do processo, o *Airflow* deve ser capaz de reexecutar sem duplicar dados no seu bucket `landing` ou `bronze`.

    - **Destaque**: Ele explica muito bem como usar o `PythonOperator` e como estruturar pastas de DAGs, o que justifica o volume que mapeamos no `docker compose` (`${PROJECT_PATH}/dags:/opt/airflow/dags`).

- ### **Apache Airflow Best Practices (Dylan Intorf et al. - Out/24)**:

    - **Onde ele entra no MVP**: Como este livro é novíssimo e foca em padrões modernos (Airflow 2.10+). Foi essencial para ajustar o Tuning de Performance.

	- **Aplicação Prática**: Ele ensinou a usar melhor os recursos da VPS. Este livro foi a base teórica para entender como os Slots de execução e as Pools funcionam para não travar a máquina quando tiver muitos projetos rodando sob a mesma infraestrutura.

	- **Destaque**: Ideal para aprender sobre o uso de **Datasets** (Data-driven scheduling), onde uma DAG de "Prata" começa automaticamente assim que a "Bronze" termina de escrever o arquivo.

---
## **2) Apache Iceberg**
O Iceberg é o **Open Table Format** que você escolheu para o coração do seu Lakehouse. Ele resolve o problema de consistência que o Data Lake "puro" possui.

- ### **Apache Iceberg: The Definitive Guide (Tomer Shiran, Jason Hughes, Alex Merced)**:

	- **Finalidade no MVP**: Como você está usando o **Dremio** e o **Nessie**, este livro é leitura obrigatória. Ele explica como o Iceberg gerencia metadados para permitir o *Time Travel* (consultar dados do passado) e o *Snapshot Isolation*.

	- **Aplicação Prática**: Ele vai te ajudar a entender o que acontece no seu bucket GCS quando o Dremio cria uma tabela. Você aprenderá a gerenciar o ciclo de vida dos manifestos e arquivos de metadados para que seu storage não fique "sujo" com arquivos órfãos.

	- **Destaque**: Escrito por fundadores da Dremio e especialistas em Iceberg, foca exatamente na stack que você montou.
---
## **3) dbt (Data Build Tool)**
O dbt é onde a "mágica" da transformação acontece. No MVP, ele será o responsável por ler a camada **Bronze** e gerar a Governança dos dados para a **Prata** e a **Ouro**.

- ### **Unlocking dbt (Dustin Dorsey, Cameron Cyr - Set/25)**:

	- **Finalidade no MVP**: Sendo um lançamento futuro (ou fresquíssimo), ele foca em padrões de implantação em nuvem. No nosso caso, ele ajudará a estruturar o *dbt* dentro do *Airflow* como um orquestrador de transformações.

- ### **Data Engineering with dbt (Roberto Zagni) & Analytics Engineering with SQL and dbt (Machado & Russa)**:

	- **Aplicação Prática**: Use o livro do Zagni para a parte pesada de engenharia (testes de dados, documentação e modularidade). O livro de Machado & Russa é excelente para o SQL aplicado ao negócio — ideal para quando você estiver modelando a Camada Ouro da sua lista de compras para o Superset.

	- **Destaque**: Como o dbt usa apenas SQL, esses livros vão me obrigam a ser um mestre em consultas performáticas.
---
## **4) DuckDB**
O DuckDB é o "canivete suíço" moderno. Ele é um banco de dados OLAP que roda em memória ou localmente, extremamente rápido para arquivos Parquet e CSV.

- ### **Getting Started with DuckDB (Aubury & Letcher)**, **DuckDB: Up and Running (Lee)** e **DuckDB in Action (Simons et al.)**:

	- **Finalidade no MVP**: Embora o Dremio seja nosso motor principal, o DuckDB pode entrar em nossa classe `IngestionManager` (*Airflow*) para fazer pré-processamentos ou validações rápidas antes mesmo de salvar o dado na Bronze.

	- **Aplicação Prática**: O livro ***DuckDB in Action*** mostra como ele se integra com Python e Pandas. O utilizamos para "limpar" a planilha de compras na memória da VPS antes de persistir o Parquet no GCS, economizando tempo de rede.

	- **Destaque**: O DuckDB consegue ler e escrever diretamente do GCS. É uma ferramenta de "baixo custo e alto impacto" para quem roda em VPS.
---
## **Resumo da Jornada de Estudo**
1. **Airflow**: Orquestra o fluxo e garante que as peças se movam.

2. **DuckDB**: Valida e limpa os dados na entrada de forma ultra-veloz.

3. **Iceberg (via Dremio/Nessie)**: Armazena com segurança, permitindo versionamento e rollback.

4. **dbt**: Transforma o dado bruto em valor de negócio (Prata -> Ouro).

Note que a biblioteca cobre todas as pontas da **Engenharia de Dados moderna**. Se seguirmos essa trilha, o MVP não será apenas um projeto de portfólio, mas uma demonstração de proficiência em ferramentas que as grandes empresas estão adotando agora.

Mas, pintou uma dúvida: *O **DuckDB** me surpreendeu, achei que ele fosse substituir o **Pandas**. Será que teremos de usar o **Pandas** para abrir o .xlsx e depois usar o **DuckDb** para converter em .csv! E outra, se o **DuckDB** é um **BD OLAP**... usá-lo apenas para converter os dados de .xlsx para csv e depois para **parquet** não seria subutilizar toda a sua potência?*
A resposta para essa percepção técnica é que tocamos em um ponto central na evolução do processamento de dados moderno: a transição do processamento focado em *Dataframes* (**Pandas**) para o processamento focado em *SQL* (**DuckDB**).

Essa questão sobre a ordem e a subutilização faz todo o sentido. Vamos analisar por que o DuckDB é muito mais que um conversor e como ele se encaixa no MVP:

## **1. DuckDB vs. Pandas: O fim da "subutilização"**
Usar o *DuckDB* apenas para converter formatos seria como usar uma Ferrari para ir à padaria. Entretanto, a mágica do *DuckDB* no pipeline é que ele pode substituir o *Pandas* em quase tudo o que envolva **transformação**
- **O problema do Pandas**: Ele carrega tudo para a memória RAM. Se o arquivo .xlsx com os itens crescer, a VPS pode travar.
- **Solução com o DuckDB**: Ele consegue ler o arquivo e processar "por blocos" (streaming), sendo muito mais eficiente.
- **Dica de Ouro**: Você não precisa converter para `.csv` no meio do caminho! O **DuckDB** consegue ler o arquivo Excel (via extensão `spatial` ou `xlsx`) e gravar diretamente em *Parquet* no seu *bucket Bronze*

## **2. A Ordem de Aprendizado Sugerida.**
A ordem (Airflow -> DuckDB -> dbt -> Iceberg) é lógica, com um pequeno ajuste para alinhar com o MVP:

1. **Airflow**: É a fundação. Sem ele, você não tem automação.
2. **DuckDB**: Use-o na camada **Landing** -> **Bronze**. Em vez de *Pandas*, use *SQL* dentro do *Python* para limpar os dados brutos e salvar o *Parquet*.
3, **Apache Iceberg (via Nessie/Dremio)**: Configure a tabela na **Camada Prata**. É aqui que o dado deixa de ser um "arquivo solto" e vira uma "tabela de verdade" com controle de transação.
4. **dbt**: Por último, pois o dbt vai orquestrar a lógica de negócio **Bronze -> Prata -> Ouro**. Ele usará o Dremio (com o motor Iceberg) para processar tudo.

## **3. O "Superpoder" do DuckDB no seu MVP**
Para não subutilizar o *DuckDB*, pense nele como o seu "**Motor de Ingestão Inteligente**":
- Em vez de apenas converter, use o *DuckDB* para fazer **Data Quality imediato**.
- Exemplo: "O campo `valor` da planilha é negativo? O *DuckDB* já filtra isso antes de gravar na Bronze".
- Isso garante que sua Camada Bronze já nasça com um nível mínimo de qualidade, economizando processamento no Dremio e dbt mais tarde.

## **4. Por que manter o Dremio/Iceberg se tenho o DuckDB?**
Nos perguntamos: "*Se o DuckDB é tão bom, por que preciso do Dremio?*"
A resposta à esta pergunta não é simples, mas podemos responder da seguinte forma
- O **DuckDB** é excelente para processar o dado de um projeto localmente (um "mecanismo de execução").

- O **Dremio + Iceberg** é uma Plataforma de Governança. Ele permite que o Superset consulte os dados, que você tenha versionamento (Nessie) e que várias pessoas (ou serviços) acessem o dado simultaneamente sem corrompê-lo.

**Resumindo a resposta final: ** Use o Pandas apenas para o "ataque inicial" no arquivo `.xlsx` (se necessário), mas deixe o trabalho pesado de transformação `SQL` para o `DuckDB`.
