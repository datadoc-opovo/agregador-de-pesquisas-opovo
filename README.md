## Metodologia do Agregador de Pesquisas O POVO

Em meio às comemorações dos dois anos do [O POVO Mais](https://mais.opovo.com.br/home), a Central de Jornalismo de Dados (DATADOC) lança o [Agregador de Pesquisas do O POVO](https://mais.opovo.com.br/interativos/agregador/index.php). Ao compilar os resultados das sondagens divulgadas por diferentes institutos de pesquisa e aplicar medidas estatísticas como média e média móvel, a ferramenta permite avaliar as tendências apresentadas pelas pesquisas eleitorais a partir de uma perspectiva mais ampla.

Neste repositório, apresentamos a metodologia utilizada para a construção do agregador e para o cálculo dos indicadores que o compõem. Com esse notebook os leitores podem ter acesso aos dados coletados e conhecer o passo a passo seguido pela equipe da Central DATADOC para chegar aos resultados e às análises apresentadas.

---

### Fonte e coleta de dados

Coletamos na página [Consulta às pesquisas registradas](https://www.tse.jus.br/eleicoes/pesquisa-eleitorais/consulta-as-pesquisas-registradas), do TSE, as principais informações das pesquisas realizadas pelos oito institutos que formam a base de pesquisas do Agregador **O POVO**. Em seguida, buscamos os resultados de cada pesquisa em portais oficiais e páginas de notícias.

Em nosso universo de pesquisas exploradas, consideramos as intenções de voto para os cargos majoritários (Senado, Governo e Presidência) de pesquisas estimuladas em todo o Brasil em seus múltiplos cenários para o 1º e 2º turno, como também a rejeição dos candidatos e a avaliação e aprovação do Presidente / Governo Federal.

---

### Bases de dados

Na própria ferramenta, além de visualizar graficamente os dados ao simular os diversos cenários possíveis, o leitor pode baixar os arquivos criados a partir da coleta realizada pela Central DATADOC junto aos oito institutos de pesquisa. Os dados estão disponíveis — com atualização frequente — em formato aberto (.csv), em duas bases: Pesquisas Registradas e Resultados Divulgados.

- [Pesquisas Registradas](https://docs.google.com/spreadsheets/d/e/2PACX-1vSN4JjB4ffRqPCIFJxCOuprBdJsxZrfWGGBgu1IeMKq4QX7skPXUKdX4PaJEkVncM7DFMXb07N0DrJ_/pubhtml?gid=0&single=true)
- [Resultados Divulgados](https://docs.google.com/spreadsheets/d/e/2PACX-1vSMclxJFkaSFATTGL7WF0OfuNX5xD5TEgxFwqhFcwKB217xkTLZ33wIDPSHCRPT2-WJP1JT9uJAxLVJ/pubhtml?gid=0&single=true)

---

### Metodologia

**Institutos de Pesquisa**
A base de pesquisas do Agregador **O POVO** é formada por oito institutos: Datafolha, Ipec, Quaest, Ipespe, Atlas Político, DataPoder, Ideia e FSB. Eles foram selecionados a partir de três critérios. O primeiro deles é a nota obtida no [Barômetro do Poder de abril](https://www.infomoney.com.br/wp-content/uploads/2022/05/INF_barometro_35_Abril2022.pdf), um relatório elaborado pela [Infomoney](https://www.infomoney.com.br/) a partir da consulta com 10 casas de análise de risco político e quatro conhecidos consultores independentes.

O segundo foi a frequência de pesquisas nacionais realizadas, de modo a garantir um volume de pesquisas considerável. Por fim, a escolha considerou a diversidade de metodologias aplicadas (presencial por fluxo e residencial; telefônica e por internet) para incluir diferentes nuances captadas por cada um dos modelos.

**Calculando Tendências**
Para calcular as tendências expressas no agregador de pesquisas do **O POVO**, a Central seguiu os seguintes passos:

1. Definição de um cenário
2. Pesos e médias diárias
3. Média móvel ponderada

Confira cada um dos passos a seguir:

1. **Definição de um cenário**

Para executar o cálculo de tendências propriamente dito, precisamos, em primeiro lugar, definir um conjunto de resultados que serão utilizados, que serão selecionados pelo leitor: *turno, cargo, tipo de pesquisa, abrangência e/ou institutos de pesquisas*.

Utilizaremos, para este exemplo, os seguintes parâmetros: pesquisas de **Intenção de Voto para Presidente no 1º Turno**. Outros cenários, como a aprovação e avaliação do governo e a rejeição dos candidatos, seguirão os mesmos cálculos e métricas. O usuário também poderá adicionar ou retirar candidatos. Aqui, para fins ilustrativos, vamos utilizar apenas o candidato **Lula**.

**Pesquisas com o Candidato:** 

![dados_do_candidato.svg](imagens_readme/dados_do_candidato.svg)

1. **Pesos e médias diárias**

Nesta etapa, executamos os seguintes passos:

- Em dias com uma ou mais pesquisas, calculamos a **média simples** das pontuações do candidato.
- Para os dias subsequentes sem pesquisas registradas, atribuímos a pontuação do candidato no dia imediatamente anterior.
- Por fim, atribuímos um peso referente ao número de cenários testados para o candidato em cada dia. Desta forma, dias com mais pesquisas terão uma influencia superior no próximo passo.

**Exemplo**: Se hoje o candidato “A” aparece em 5 cenários e ontem apareceu em 3, a sua pontuação de hoje terá um peso 5 e a pontuação de ontem terá um peso 3 para a próxima etapa. Para os dias sem pesquisas registradas, em que a média diária segue o valor do dia anterior, atribuímos o peso 

**Gráfico dos pesos e médias diárias:** 

![grafico_media_simples.svg](imagens_readme/grafico_media_simples.svg)

**Código:**

```jsx

medias_diarias = {

	function groupBy(xs, key) {
	  return xs.reduce(function(rv, x) {
	    (rv[x[key]] = rv[x[key]] || []).push(x);
	    return rv;
	  }, {});
	};

  const pesquisas_por_dia = groupBy(pesquisas_do_candidato, 'data');

  const dias_minimo_maximo = [pesquisas_do_candidato.map(d=> d.data)[0], new Date()];

  const dias = d3.timeDay
    .range(dias_minimo_maximo[0], dias_minimo_maximo[1])
    .map(d => d)

  var medias_diarias = [];
  
	dias.forEach(dia => {

    if(pesquisas_por_dia[dia]){

      var media_diaria = d3.mean(pesquisas_por_dia[dia].map(d=> d['Pontuação (%)'])),
        peso = pesquisas_por_dia[dia].length;

      medias_diarias[dia] = {'data':dia, 'media':media_diaria,'peso':peso,'ref':'media'};

    }else if(medias_diarias[d3.timeDay.offset(new Date(dia),-1)]){

      media_diaria = medias_diarias[d3.timeDay.offset(new Date(dia),-1)].media;
      peso = 1;

      medias_diarias[dia] = {'data':dia,'media':media_diaria,'peso':peso,'ref':'dia_anterior'};

    }else{
    
		  medias_diarias[dia] = {'data':dia,'media':0,'peso':1,'ref':'inexistente'};
    }

  });

  return Object.values(medias_diarias);
}
```

1. **Média móvel ponderada**

Na etapa final do agregador calculamos a média móvel ponderada dos últimos 30 dias, considerando as médias diárias e os pesos atribuídos nos passos anteriores.

Desta forma, garantimos que os dias com mais pesquisas tenham maior relevância e que resultados muito discrepantes tenham o impacto reduzido nas linhas de tendência do agregador.

Além disso, neste período em que a frequência de pesquisas publicadas pelos institutos utilizados é baixa, uma janela de tempo de 30 dias consegue minimizar o impacto de períodos com poucas pesquisas coletadas.

**Gráfico da média móvel ponderada**

![grafico_media_movel.svg](imagens_readme/grafico_media_movel.svg)

```jsx
media_movel = {

  function media_ponderada(vetor_medias_e_pesos) {
    var somatorio = 0, pesos=0;  
    vetor_medias_e_pesos.forEach(d=>{
      somatorio += (d[0]*d[1]);
      pesos+= d[1];
    });
    return somatorio/pesos;
  }

  var df = [];
  medias_diarias.forEach((d,i)=> {
    const date = d.data;

    var mediaMovel = 0;
    if(i>controle_mMovel){
        mediaMovel = media_ponderada(medias_diarias.slice(i-controle_mMovel, i).map(m => [m.media, m.peso]));
        df.push({'data':date,'media':d.media,'mMovel':mediaMovel,'peso':d.peso,'ref':d.ref});
      }
      else{      
        df.push({'data':date,'media':d.media,'mMovel':NaN,'peso':d.peso,'ref':d.ref});
      }
  });
  return df;
}
```

---

### Perguntas frequentes

**1. O que é o agregador de pesquisas?**

Um agregador de pesquisas é uma ferramenta que possibilita uma perspectiva ampla das tendência em diferentes cenários eleitorais, agregando, por meio de um conjunto métricas estatísticas, o resultado expresso em múltiplas sondagens, com metodologias, amostras, contratados e contratantes distintos.

Com um agregador, o leitor pode visualizar as tendências aferidas pelas diferentes pesquisas ao longo do tempo, de modo que resultados discrepantes e fenômenos efêmeros tenham um impacto minimizado no contexto geral. Vale pontuar também que um agregador não se resume às pesquisas de intenção de voto em eleições e pode ser aplicado em diferentes cenários e situações, como rejeição de candidatos, aprovação/avaliação de governos, entre outros.

**2. O agregador compara metodologias?**

Não. A função de um agregador não é comparar as características amostrais e metodológicas do seu universo de pesquisas, mas traçar uma linha de tendência que possa representar de maneira ampla o conjunto dos diferentes resultados em um dado cenário.

Para tanto, um agregador buscar priorizar períodos com mais pesquisas e suavizar a interferência de resultados muito discrepantes, de lacunas temporais com poucos registros e de fenômenos breves e ocasionais.

**3. Como essa ferramenta pode me ajudar a entender o atual cenário eleitoral?**

Frequentemente, diversos institutos de pesquisas realizam sondagens que buscam retratar a opinião pública sobre diferentes assuntos. Tais pesquisas se sustentam em diferentes metodologias e podem refletir ou não os anseios da sociedade em um dado momento.

Em períodos eleitorais, por exemplo, pesquisas são publicadas cotidianamente, buscando evidenciar a percepção da sociedade sobre os candidatos e suas respectivas movimentações eleitorais. Essa é uma dinâmica que se intensifica gradativamente à medida que nos aproximamos das eleições.

Entretanto, alguns pontos precisam ser observados ao analisar os resultados. O primeiro deles é que cada pesquisa retrata o momento em si, e não uma tendência eleitoral abrangente. Além disso, com diferentes critérios e metodologias, as pesquisas podem possuir diferentes vieses e apresentar resultados completamente divergentes.

Neste contexto, uma ferramenta como o Agregador de Pesquisas do **O POVO**, que congrega a diversidade de metodologias aplicadas no Brasil e executa um conjunto claro e consistente de métricas estatísticas, pode contribuir para uma análise ampla e consistente das tendências eleitorais.

---

### A Central DATADOC

A Central de Jornalismo de Dados do **O POVO** (DATADOC) alia tecnologia e técnicas diversas de análises de dados para produzir um jornalismo de precisão para que você forme sua opinião com segurança. Nosso objetivo é fazer com que todos tenham acesso aos dados utilizados nas notícias que produzimos.

A DATADOC é composta por uma equipe de três jornalistas (sendo uma infografista), uma desenvolvedora front-end e um cientista da computação que coletam, enriquecem e disponibilizam as bases e códigos de cada reportagem para um jornalismo transparente e baseado em evidências.

---

**🔥📰👩🏻‍💻 Se você gostou do nosso material, apoie assinando o OP+ e acompanhando o nosso trabalho.**

**📝📨 Para feedback, dúvidas ou sugestões: [datadoc@opovodigital.com](mailto:datadoc@opovodigital.com)**

---

👩🏻👩🏾👩🏿 Confira também outras produções recentes da central DATADOC: A reportagem ***Ceará ocupa a 24º posição nacional, em representatividade de gênero na política*** mostrou que, em relação a representatividade de gênero, o Ceará fica à frente apenas do Mato Grosso, Mato Grosso do Sul e Goiás. A matéria está [disponível no O POVO+](https://bit.ly/38qHm11).