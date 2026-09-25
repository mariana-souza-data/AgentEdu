# AgentEdu Score

O AgentEdu Score é um modelo explicável de crédito educacional, construído inteiramente sobre dados públicos do INEP e da RAIS. Ele nasceu de uma pergunta simples: dá pra financiar estudante sem exigir fiador, e ainda assim ser sustentável pro banco que empresta? Este repositório reúne a prova de conceito completa, com notebook, apresentação, roteiro e documentação técnica.

Trabalho de Conclusão de Curso em Ciência de Dados / Inteligência Artificial, desenvolvido por Mariana de Lima Souza.

## O problema

Hoje o financiamento estudantil no Brasil vive entre duas portas ruins. O Fies é público, mas está sob uma crise estrutural de inadimplência. Fora dele, o financiamento privado normalmente exige fiador ou garantia real, o que acaba excluindo justamente quem mais precisa de crédito. E o critério de elegibilidade do próprio Fies, baseado em renda per capita e nota do ENEM, não avalia o risco real da operação: não estima a chance de o aluno concluir o curso nem a capacidade de pagamento que ele vai ter depois de formado.

## A proposta

O AgentEdu Score parte da mesma renda per capita que o Fies já usa, mas soma dois sinais que o programa ignora hoje: a evasão real do curso escolhido (INEP, corrigida por uma janela de observação estendida, pra separar quem realmente evadiu de quem só se formou atrasado) e a empregabilidade da área, medida pelo salário de nível inicial na RAIS. O resultado é um score de 0 a 100 pontos, com a composição inteira auditável, então dá pra ver exatamente de onde veio cada ponto.

A parte de negócio segue a mesma lógica de transparência: em vez de inventar um mecanismo novo, ela reaproveita o FGEDUC, o fundo garantidor que já sustenta o Fies hoje, e propõe tornar a cobertura dele diferenciada por risco. Score mais baixo não vira "peça um fiador", vira "o fundo cobre uma fatia maior". Em nenhuma faixa de score existe exigência de garantia pessoal do aluno.

## O que tem em cada pasta

O notebook está em `notebook/`, junto com o script Python que o gera. É onde toda a lógica roda de fato: consulta às bases públicas, cálculo da evasão de janela estendida, indicador de empregabilidade, calibração do peso por regressão linear e a função `calcular_score()`.

Em `pptx_final/` está a apresentação de 20 minutos (18 slides), com o script que a monta. Em `roteiro_final/` estão o roteiro falado, slide a slide, e um guia de conteúdo pra montagem dos slides. E em `documentacao_final/` está o relatório técnico completo, cobrindo arquitetura, metodologia, o código explicado e os resultados.

## De onde vêm os dados

Os dados de evasão vêm do Censo da Educação Superior do INEP, e os de empregabilidade vêm da RAIS, ambos acessados via [Base dos Dados](https://basedosdados.org), que já disponibiliza essas bases tratadas no BigQuery. É tudo dado público e agregado por curso, instituição e ano; em nenhum momento o modelo usa dado pessoal ou identificável de um candidato real. O recorte da prova de conceito é o estado de Pernambuco, com cinco cursos de graduação (Administração, Engenharia Civil, Direito, Pedagogia e Ciências Sociais) e coortes de ingresso entre 2010 e 2012.

## Como rodar

Pra rodar o notebook você vai precisar do Python com `basedosdados`, `pandas`, `numpy` e `scikit-learn` instalados, além de uma conta Google Cloud com um billing project configurado (o notebook usa isso pra consultar o BigQuery através da biblioteca `basedosdados`). Se estiver no Colab, basta abrir o notebook, autenticar sua conta Google e trocar o `PROJECT_ID` pelo seu.

```bash
pip install basedosdados pandas numpy scikit-learn
```

Depois é só rodar as células em ordem. A lógica de evasão de janela estendida está na seção 6, o indicador de empregabilidade na seção 8, a calibração por regressão na seção 9, a função de score na seção 10, e o teste contra as sete personas na seção 11.

Link do Colab: *(cole aqui o link de compartilhamento do seu notebook)*

## Resultados

O modelo foi testado contra sete personas fictícias, cobrindo os cinco cursos do recorte e dois casos de fronteira propositais: renda muito baixa combinada com curso de alta empregabilidade, e renda alta combinada com o curso de maior evasão real. Em todos os casos o financiamento concedido acompanhou coerentemente o risco estimado, e mesmo os cursos de menor retorno (Pedagogia, Ciências Sociais) receberam financiamento entre 70% e 90%, sem qualquer exigência de fiador. Os números completos, com a tradução de score pra faixa de financiamento, estão detalhados na documentação técnica.

## Limitações

Vale deixar claro, com a mesma transparência que o modelo pede pra si mesmo: o desempenho acadêmico ainda é um dado simulado nesta POC, sem conexão com histórico escolar real. Ciências Sociais tem uma amostra pequena na RAIS (15 vínculos), então esse número deve ser lido com ressalva, e o próprio script já avisa isso automaticamente sempre que acontece. O recorte geográfico cobre só Pernambuco, e a regressão que calibra o peso de empregabilidade usa apenas cinco pontos de dado, o que a deixa estatisticamente fraca pra qualquer inferência causal mais forte. E o modelo de negócio, por enquanto, ainda não foi simulado financeiramente contra um portfólio maior de scores: é uma proposta conceitual apoiada na estrutura já validada do FGEDUC.

## Ética, privacidade e vieses

Como já foi dito, todo dado usado é público e agregado, sem nenhuma informação pessoal de candidato real em nenhuma etapa. As personas de teste são fictícias, criadas só pra validar o comportamento do score.

Existe, sim, um viés estrutural conhecido: o componente de empregabilidade tende a penalizar cursos de retorno salarial menor, como Pedagogia e Ciências Sociais. Isso não é um efeito colateral ignorado, é exatamente por causa desse viés que o modelo de negócio nunca permite exigência de fiador em nenhuma faixa de score, por mais baixa que seja. O risco mais alto é absorvido pelo fundo garantidor, não empurrado pro aluno. E a amostra pequena de RAIS em Ciências Sociais é sinalizada automaticamente pelo script sempre que aparece, justamente pra evitar que uma decisão de crédito se apoie num número estatisticamente frágil sem essa ressalva visível.

## Referências

Base dos Dados, dados públicos tratados, disponível em basedosdados.org. Lei nº 10.260/2001, que institui o Fies. Censo da Educação Superior do INEP, microdados por curso (2010 a 2012). RAIS, Ministério do Trabalho e Emprego, microdados de vínculos, ano-base 2022. Google BigQuery, disponível em cloud.google.com/bigquery. Pedregosa et al., "Scikit-learn: machine learning in Python", JMLR, v. 12, 2011. McKinney, "Data structures for statistical computing in Python", Proc. 9th Python in Science Conf., 2010.
