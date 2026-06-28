# Módulo 3: Anotação Genômica – Da Sequência à Função Biológica

A anotação genômica representa o ponto de inflexão em um projeto de bioinformática: é o processo pelo qual a sequência linear de nucleotídeos (ATCG) é traduzida em **conhecimento biológico funcional**. Se a montagem responde à pergunta *"qual é a sequência?"*, a anotação responde a *"o que esses genes fazem?"* e *"como este organismo vive, interage e se adapta?"*. Para genomas bacterianos e arqueanos, a anotação envolve a identificação precisa de genes codificadores de proteínas (CDS), RNAs não codificantes, elementos móveis e regiões regulatórias, seguida da atribuição de funções com base em bancos de dados curados e em ferramentas de homologia.

Atualmente, a anotação automatizada de alta qualidade (com ferramentas como **Prokka** e **Bakta**) é capaz de processar um genoma bacteriano completo em poucos minutos, gerando arquivos padronizados (GFF3, GBK, GenBank) que são a base para análises comparativas, filogenéticas e funcionais. A tendência moderna é a integração de múltiplas fontes de evidência (ab initio, homologia, perfil HMM e estrutura secundária) para maximizar a acurácia e a cobertura da anotação.

---

## 1. Fases da Anotação Genômica

A anotação genômica é classicamente dividida em duas grandes fases complementares: a **anotação estrutural** (identificação dos elementos) e a **anotação funcional** (atribuição de significado biológico).

### 1.1. Anotação Estrutural

A anotação estrutural tem como objetivo **localizar e delimitar** todos os elementos genômicos ao longo da sequência de DNA. Isso inclui:

- **Genes codificadores de proteínas (CDS – *Coding DNA Sequence*)**: A maioria dos genes bacterianos não possui íntrons, o que simplifica a predição, mas ainda exige algoritmos sofisticados para identificar *Open Reading Frames* (ORFs) e sítios de ligação ao ribossomo (RBS – sequência de Shine-Dalgarno).
- **RNAs ribossômicos (rRNAs)**: Subunidades 5S, 16S e 23S, essenciais para a estrutura e função do ribossomo.
- **RNAs transportadores (tRNAs)**: Fundamentais para a tradução da mensagem genética em proteínas.
- **Genes hipotéticos**: ORFs com potencial codificante, mas sem homologia conhecida – representam uma parcela significativa em genomas novos e são um vasto campo para descoberta funcional.
- **Elementos móveis**: Sequências de inserção (IS), transposases, integras e fagos, importantes para a dinâmica genômica e evolução.
- **(Opcional) Regiões promotoras e terminadoras**: Sítios de início e fim da transcrição, preditos por modelos de *hidden Markov* (HMM).

**Ferramentas representativas ativamente mantidas**:

| Ferramenta | Tipo | Característica principal |
|:---|:---|:---|
| **Prokka** | *Pipeline* completo (wrapper) | Padrão ouro para procariotos; integra Prodigal, Barrnap, tRNAscan-SE e Blast+. |
| **Bakta** | *Pipeline* completo (moderno) | Sucessor conceitual do Prokka; melhor para detecção de plasmídeos e anotação de genes de resistência; suporta bases de dados mais atualizadas. |
| **PGAP** (NCBI) | *Pipeline* oficial | Usado pelo NCBI para submissão de genomas ao GenBank; altamente padronizado e rigoroso. |
| **GeneMarkS** | *Ab initio* | Utiliza modelos de Markov não-homogêneos; muito preciso para bactérias, especialmente em genomas com composição GC extrema. |
| **Prodigal** | *Ab initio* | Motor de predição de CDS utilizado internamente pelo Prokka; baseia-se em programação dinâmica e modelos de Markov de ordens variáveis. |
| **Barrnap** | RNAs ribossômicos | Rápido e específico para rRNA 5S, 16S e 23S; utiliza HMMs curados. |
| **tRNAscan-SE** | tRNAs | Padrão ouro; detecta tRNAs por estrutura em *cloverleaf* (folha de trevo) e sequência anticódon, com alta sensibilidade. |

### 1.2. Anotação Funcional

Uma vez que os elementos estruturais estão delimitados, a **anotação funcional** atribui **funções biológicas conhecidas** a esses elementos, por comparação com bancos de dados públicos curados. Essa etapa é fundamental para transformar uma lista de ORFs em um mapa metabólico e fisiológico do organismo. O princípio subjacente é a **transferência de função por homologia** (*guilt‑by‑association*): sequências similares tendem a ter funções similares.

A anotação funcional inclui a atribuição de:

- **Nome da proteína** (ex: "DNA polymerase III subunit beta").
- **Vias metabólicas** (via KEGG, MetaCyc) – posicionando o gene em rotas bioquímicas.
- **Famílias de proteínas** (Pfam, TIGRFAM, COG) – agrupando por domínios evolutivos.
- **Termos de Ontologia Gênica (GO)** – descrevendo função molecular, processo biológico e componente celular.
- **Genes de resistência a antimicrobianos (ARGs)** – via AMRFinderPlus ou CARD.
- **Fatores de virulência** – via VFDB ou Victors.

**Ferramentas e bancos de dados representativos (ativos e atualizados)**:

| Ferramenta | Banco/Database | Objetivo principal |
|:---|:---|:---|
| **InterProScan** | InterPro (Pfam, SMART, PROSITE, etc.) | Identificação de domínios proteicos e assinaturas funcionais; integra múltiplos bancos. |
| **HMMER** | Pfam, TIGRFAM | Busca por perfil de *hidden Markov models* (HMM), altamente sensível para famílias distantes. |
| **eggNOG-mapper** | eggNOG (COG, KEGG, GO) | Anotação funcional ortóloga, vias metabólicas e termos GO; atualizado regularmente. |
| **BLASTp** | NR (NCBI), UniProt (Swiss-Prot) | Busca por homologia sequencial contra bancos de proteínas curados e não curados. |
| **AMRFinderPlus** | NCBI AMR | Detecção de genes de resistência a antibióticos e mutações associadas; padrão do CDC/NCBI. |
| **RAST** (online) | SEED | Pipeline completo de anotação com curadoria manual parcial; útil para iniciantes, mas limitado em termos de customização. |

> **Nota conceitual**: A anotação funcional baseia‑se no princípio da **homologia** (similaridade de sequência sugere similaridade de função) e da **ortologia** (genes ortólogos compartilham a mesma função em diferentes espécies). O uso de perfis HMM (como Pfam) é mais sensível que BLAST para detectar domínios distantes, pois modela a variabilidade aceitável dentro de uma família proteica.

---

## 2. Etapas Detalhadas da Anotação Estrutural

A seguir, detalhamos as etapas práticas mais importantes para a anotação estrutural de genomas procariotos.

### 2.1. Previsão de Sequências de DNA Codificantes (CDS)

A predição de CDS é o coração da anotação estrutural. Em bactérias, a ausência de íntrons torna o processo mais direto, mas ainda exige a correta identificação do **quadro de leitura** (*reading frame*) e do **sítio de início da tradução** (RBS – sequência de Shine-Dalgarno). Algoritmos modernos como o Prodigal utilizam **Modelos de Markov Interpolados** (IMM) que se ajustam dinamicamente ao conteúdo GC do genoma, distinguindo regiões codificantes (com viés de uso de *codons*) de regiões não codificantes.

Ferramentas como **Prokka** utilizam internamente o **Prodigal** (*Prokaryotic Dynamic Programming Genefinding Algorithm*), que é capaz de detectar genes completos, fragmentos e *overlaps* gênicos (comuns em procariotos devido à compactação genômica).

**Exemplo de um produto gênico anotado (formato GFF3)**:

```
contig_1  Prodigal  CDS  105  978  .  +  0  ID=cds00001;product=hypothetical protein
```

**Interpretação das colunas GFF3** (formato padronizado para troca de dados genômicos):

| Coluna | Campo | Exemplo | Significado |
|:---|:---|:---|:---|
| 1 | *seqid* | `contig_1` | Identificador da sequência (contig/scaffold) onde o gene está localizado. |
| 2 | *source* | `Prodigal` | Ferramenta/algoritmo que gerou a predição. |
| 3 | *type* | `CDS` | Tipo de elemento (CDS, rRNA, tRNA, gene, etc.). |
| 4 | *start* | `105` | Posição inicial da coordenada no contig (base 1, inclusive). |
| 5 | *end* | `978` | Posição final da coordenada (inclusive). |
| 6 | *score* | `.` | Pontuação de confiança (`.` quando não disponível ou não aplicável). |
| 7 | *strand* | `+` | Fita positiva (plus) ou negativa (minus). |
| 8 | *phase* | `0` | Fase de leitura (0, 1 ou 2) – indica o deslocamento do primeiro *codon* completo em relação à primeira base; 0 significa que o gene inicia no primeiro nucleotídeo de um *codon*. |
| 9 | *attributes* | `ID=...;product=...` | Atributos no formato `chave=valor`; `ID` é obrigatório; `product` contém a descrição funcional (se disponível). |

> **Observação**: A presença frequente de `product=hypothetical protein` é comum em genomas novos e indica que, embora a estrutura do gene tenha sido predita com alta confiança, nenhuma função conhecida pôde ser atribuída por homologia. Estes representam um vasto "oceano de diversidade funcional" ainda inexplorado e são alvos frequentes de estudos de caracterização experimental.

### 2.2. Identificação de RNAs não Codificantes

A identificação de RNAs não codificantes é uma etapa essencial da anotação genômica, especialmente para a completude funcional (tRNAs) e para a construção de árvores filogenéticas (usando o 16S rRNA). Os principais tipos são:

- **rRNAs** (RNA ribossômico): subunidades 5S, 16S e 23S.
- **tRNAs** (RNA transportador): responsáveis pelo carregamento de aminoácidos.
- **sRNAs** (pequenos RNAs reguladores): envolvidos na regulação pós-transcricional, detectados por modelos estruturais.

#### 2.2.1. rRNAs (RNA Ribossômico) – O que são?

São componentes essenciais do ribossomo, a maquinaria de síntese proteica. Em procariotos, os principais genes rRNA são organizados em operons (geralmente na ordem 16S – 23S – 5S) e são altamente conservados, sendo amplamente usados como marcadores filogenéticos (especialmente o 16S rRNA).

**Ferramentas principais para predição de rRNA (modernas e ativamente mantidas)**:

---

**1. Barrnap** (*Bacterial Ribosomal RNA Predictor*) – **Abordagem Padrão**

Ferramenta rápida, leve e confiável para detecção de rRNAs em genomas de bactérias e arqueias. Utiliza modelos HMM específicos para cada subunidade (5S, 16S, 23S) treinados em sequências de alta qualidade. É a escolha padrão integrada ao Prokka e Bakta, sendo amplamente utilizada em *pipelines* de alto rendimento.

**Exemplo de uso**:
```bash
barrnap --kingdom bac <genome.fasta> > rRNA.gff
```
- `--kingdom bac`: específico para bactérias (use `arc` para arqueias).
- A saída é gerada no formato `.gff`, que pode ser integrado diretamente a *pipelines* de anotação ou visualizado em ferramentas como Artemis/IGV.

**Fonte**: [https://github.com/tseemann/barrnap](https://github.com/tseemann/barrnap)

> **Nota sobre ferramentas legadas**: Ferramentas mais antigas, como o **RNAmmer**, encontram-se atualmente descontinuadas e sem suporte ativo (o serviço online e as atualizações foram encerrados pelo desenvolvedor). Para novos projetos, recomenda‑se fortemente o uso de **Barrnap** por sua simplicidade, velocidade e manutenção ativa. Para análises que exigem máxima sensibilidade em RNAs estruturais ou a detecção de múltiplas classes (sRNAs, riboswitches), a abordagem baseada em modelos de covariância (Infernal + Rfam), descrita a seguir, é a mais adequada.

---

**2. Infernal + Rfam** (Abordagem de Modelos de Covariância) – **Máxima Sensibilidade**

Esta é a abordagem mais **robusta e sensível** para detecção de RNAs estruturais, pois o **Infernal** (INFERence of RNA Alignment) utiliza **Modelos de Covariância** (CM – *Covariance Models*). Diferentemente dos HMMs que modelam apenas a sequência, os CMs modelam simultaneamente a conservação da sequência primária e a **estrutura secundária** (pares de bases e *loops*), permitindo detectar RNAs mesmo quando a identidade de sequência é baixa, mas a estrutura é conservada. O banco **Rfam** fornece os modelos CM curados para milhares de famílias de RNA (incluindo rRNAs, tRNAs, sRNAs, riboswitches e elementos reguladores).

**Exemplo de uso com Infernal + Rfam**:
```bash
cmscan --tblout rRNA.tblout --fmt 2 --noali -o rRNA.out Rfam.cm genome.fasta
```
- `Rfam.cm`: arquivo com o banco de modelos de covariância do Rfam (deve ser baixado previamente do site do Rfam).
- `--tblout`: arquivo tabular com os *hits* encontrados, facilitando a análise computacional.
- `--fmt 2`: formato de saída para processamento em *pipelines*.
- `--noali`: suprime o alinhamento detalhado (economiza espaço em disco).
- Os resultados incluem vários tipos de RNAs, com alta especificidade e sensibilidade, sendo a ferramenta de escolha para genomas não-modelo ou para a curadoria fina de elementos reguladores.

**Fonte**: Infernal – [http://eddylab.org/infernal/](http://eddylab.org/infernal/)

> **Recomendação prática para *pipelines* modernos**:
> - Para **rRNAs e tRNAs** em genomas bacterianos de rotina: utilize **Barrnap** (rRNA) + **tRNAscan-SE** (tRNA) – rápidos e confiáveis.
> - Para **detecção de sRNAs, riboswitches e outras classes não canônicas**: complemente com **Infernal + Rfam**, que é o padrão ouro em estudos de RNA não codificante.
> - Ferramentas legadas como RNAmmer e RNAmmer-based serviços *online* não são recomendadas para novos fluxos de trabalho.

---

## 3. Considerações Finais e Boas Práticas em Anotação

A anotação genômica não é um processo estático, mas sim uma **inferência probabilística** que melhora continuamente com a atualização dos bancos de dados. Para garantir a qualidade, a reprodutibilidade e a conformidade com os padrões internacionais, recomenda‑se:

1. **Padronização de saída**: Sempre gere arquivos nos formatos GFF3, GenBank (.gbk) e FASTA de proteínas. O formato GFF3 é o padrão para troca de dados entre ferramentas; o GenBank é exigido para submissão ao NCBI.
2. **Ambientes isolados**: Utilize **Conda/Mamba** ou **Docker/Singularity** para gerenciar as versões das ferramentas, garantindo que a anotação seja reproduzível em diferentes máquinas e ao longo do tempo.
3. **Curadoria manual assistida**: Em genomas de interesse clínico ou biotecnológico, revise manualmente genes de resistência (AMR), virulência e vias metabólicas principais utilizando visualizadores como **Artemis** ou **IGV**, especialmente em regiões com múltiplas anotações conflitantes.
4. **Atualização de bancos**: Mantenha os bancos de dados (Pfam, KEGG, NCBI NR, Rfam) atualizados, pois a anotação funcional é diretamente dependente da qualidade e da abrangência das bases de referência. Ferramentas como `eggNOG-mapper` e `InterProScan` já possuem mecanismos de atualização automática.
5. **Validação cruzada**: Compare os resultados de diferentes *pipelines* (ex: Prokka vs. Bakta) para identificar discrepâncias, especialmente em regiões repetitivas, ricas em GC ou com alta densidade gênica. O Bakta, por exemplo, utiliza bases de dados mais recentes para resistoma e mobiloma, podendo complementar o Prokka.

Com a conclusão da anotação estrutural e funcional, seu genoma bacteriano ou arqueano estará pronto para análises de nível superior: **genômica comparativa** (pan‑genoma, core‑genoma), **filogenética** (baseada em genes ortólogos, como o core‑genoma), **identificação de vias metabólicas** (reconstrução metabólica *in silico*) e **estudos de evolução molecular** (detecção de seleção positiva e rearranjos).
```
