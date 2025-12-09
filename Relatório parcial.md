Desenvolvimento de framework de métricas para monitoramento e validação independente de modelos internos (IRB)

### **1\. Resumo Executivo (obrigatório)**

Este relatório parcial apresenta os avanços do projeto “Desenvolvimento de framework de métricas para monitoramento e validação independente de modelos internos (IRB)”. Nesta etapa, as atividades concentraram-se no levantamento de referências regulatórias e acadêmicas, na sistematização de métricas de monitoramento de performance e risco de modelos e na elaboração do capítulo de monitoramento do caderno de candidatura IRB. Os resultados obtidos contribuem para aprimorar a governança de modelos, aumentar a aderência às exigências regulatórias e estabelecer as bases metodológicas para o desenvolvimento futuro de uma ferramenta automatizada de monitoramento.

### **1.2 Resumo Técnico (obrigatório)**

Do ponto de vista técnico, o trabalho consistiu em: (i) revisão de normas regulatórias e boas práticas internacionais sobre monitoramento de modelos internos; (ii) mapeamento e classificação de métricas de desempenho (discriminação, calibração, estabilidade, acurácia e backtesting) adequadas ao contexto dos portfólios IRB do banco; (iii) proposição de um framework de monitoramento que organiza indicadores, frequência de cálculo e recortes analíticos; e (iv) redação do capítulo de monitoramento do caderno de candidatura. Essa etapa estrutura os requisitos funcionais e informacionais necessários para uma futura ferramenta de apoio à validação independente, alinhando a arquitetura de métricas às demandas regulatórias e internas de governança.

### **2\. Introdução (obrigatório)**

Os modelos internos utilizados para cálculo de capital regulatório sob a abordagem IRB exigem rotinas robustas de desenvolvimento, validação e monitoramento contínuo, em linha com as orientações de órgãos reguladores nacionais e internacionais. Nesse contexto, o projeto tem como objetivo desenvolver um framework de métricas e rotinas de monitoramento que fortaleça a qualidade e a eficiência da validação independente dos modelos IRB do banco, servindo também como base para o desenho de uma futura ferramenta automatizada.

Esta etapa do projeto concentrou-se na consolidação da base conceitual e metodológica necessária para esse framework, com foco na sistematização de métricas de monitoramento e na sua formalização no caderno de candidatura IRB, em particular no capítulo dedicado ao acompanhamento de performance e risco dos modelos.

### **3\. Metodologia (obrigatório)**

A metodologia adotada nesta fase combinou:  
 **(i) Revisão normativa e bibliográfica**, abrangendo documentos de supervisores bancários, diretrizes internacionais de validação e monitoramento de modelos e literatura técnica sobre métricas de desempenho e risco;  
 **(ii) Mapeamento de práticas internas**, por meio de levantamento das métricas atualmente utilizadas nas rotinas de desenvolvimento, validação e monitoramento de modelos;  
 **(iii) Análise comparativa de métricas**, com avaliação de vantagens, limitações e aderência às exigências regulatórias e às características dos portfólios de crédito do banco;  
 **(iv) Estruturação de um framework de monitoramento**, organizando indicadores, periodicidade, recortes mínimos de análise e responsabilidades;  
 **(v) Redação do capítulo de monitoramento do caderno de candidatura IRB**, incorporando as definições metodológicas consolidadas ao documento formal do projeto.

### **4\. Resultados (obrigatório)**

Nesta etapa do projeto foram consolidados os primeiros resultados aplicados ao monitoramento de modelos internos, com foco em dois artefatos principais: (i) a especificação das métricas e rotinas de acompanhamento do modelo de LGD e (ii) a definição do framework de monitoramento do modelo de FCC. Essas entregas estruturam, de forma padronizada, períodos de desenvolvimento e observação, granularidade, métricas de erro e periodicidades de cálculo e reporte, servindo como base para a futura implementação em ferramenta.

**Modelo de LGD**  
 Para o modelo de LGD, foi definida como métrica central de monitoramento a comparação sistemática entre os valores estimados e os valores observados de LGD ao longo do tempo. O período de desenvolvimento do modelo foi delimitado entre janeiro de 2015 e dezembro de 2022, enquanto o período de observação para monitoramento foi estabelecido entre dezembro de 2022 e dezembro de 2024\. A avaliação é conduzida com separação por nível de risco, permitindo identificar eventuais desvios de performance por faixa de rating e segmentar o acompanhamento de forma mais aderente ao perfil de risco das exposições. A periodicidade proposta para o cálculo das métricas é mensal, com consolidação e apresentação dos resultados em base trimestral, de modo a equilibrar granularidade analítica e estabilidade das séries.

**Modelo de FCC**  
 No caso do modelo de FCC, o período de observação considerado para o monitoramento foi definido entre janeiro de 2015 e abril de 2025\. O framework proposto contempla a comparação entre o FCC estimado e o FCC observado, bem como entre o EAD estimado e o EAD observado, de forma a capturar eventuais desvios de modelagem tanto na projeção de exposição quanto na conversão efetiva de limites em saldo devedor. Como métrica de erro principal foi adotado o WAPE (Weighted Absolute Percentage Error), o que permite ponderar adequadamente os desvios em função do volume de exposição de cada segmento. A granularidade de análise está alinhada às famílias de produto utilizadas no desenvolvimento do modelo – conta corrente ou LIS rotativos, cartão de crédito rotativo e outros limites –, possibilitando identificar comportamentos diferenciados por tipo de produto. Assim como no modelo de LGD, a periodicidade proposta de monitoramento é mensal, com apresentação trimestral para fins de governança e reporte.

De forma agregada, esses resultados materializam um framework de métricas e rotinas de monitoramento aplicável aos modelos IRB do banco, já incorporado ao capítulo de monitoramento do caderno de candidatura e pronto para ser traduzido em requisitos funcionais da futura ferramenta de apoio à validação independente.

