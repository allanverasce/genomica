# Módulo 2: Montagem do Genoma – Da Fragmentação à Continuidade Estrutural

A montagem *de novo* representa um dos problemas computacionais mais desafiadores e fascinantes da bioinformática. Dado um conjunto massivo de fragmentos curtos (*reads*) gerados pelo sequenciador, o objetivo é reconstruir a arquitetura original do genoma — uma tarefa análoga a montar um quebra-cabeça de bilhões de peças sem dispor da imagem de referência (o genoma de referência). Neste módulo, abordaremos a montagem por *grafos de Bruijn*, a execução prática do montador SPAdes e a validação estatística da qualidade da montagem utilizando métricas de **contiguidade** (QUAST) e **completude evolutiva** (BUSCO).

---

## 2.1 Fundamentos Teóricos da Montagem *De Novo*

### O Paradigma do Grafo de Bruijn

Montadores modernos para dados de *short-reads* (Illumina) baseiam-se na construção de um **grafo de Bruijn** (de Bruijn Graph). A lógica fundamental é:

1.  **Fragmentação em *k-mers***: Cada *read* (ex: 150 pb) é decomposto em todas as suas subsequências contíguas de comprimento fixo \(k\) (ex: \(k=21\) a \(127\)).
2.  **Nós e Arestas**: Cada *k-mer* único torna-se um **nó** no grafo. Dois nós são conectados por uma **aresta** direcionada se houver uma sobreposição de \((k-1)\) bases entre eles (ex: `ACG` → `CGT`).
3.  **Caminhamento (*Traversal*)**: A sequência original do genoma corresponde a um caminho específico através deste grafo. Repetições genômicas (sequências repetidas) criam *bifurcações* (*branches*) ou *ciclos*, que são os principais desafios para a montagem.

<img src="imgs/grafo.png" alt="Grafos de Bruijn" width="800" height="600" />
*Representação esquemática de um grafo de Bruijn montado a partir de *k-mers*. Cada nó é uma sequência de comprimento \(k\), e as arestas indicam sobreposições \((k-1)\). Caminhos lineares representam regiões não repetitivas do genoma.*

### A Escolha do Tamanho do *k-mer* (Trade-off Biológico)

O montador SPAdes emprega uma estratégia **multi-k-mer** (testando vários \(k\) simultaneamente) para maximizar a acurácia, mas entender o parâmetro \(k\) é crucial:

| Valor de \(k\) | Vantagens | Desvantagens |
|:---|:---|:---|
| **Baixo (ex: 21)** | Alta sensibilidade em regiões de baixa cobertura. Montagem mais contígua se a profundidade for baixa. | Menor especificidade; mais propenso a *bifurcações* falsas em regiões repetitivas. |
| **Alto (ex: 127)** | Alta especificidade; resolve repetições longas com mais eficácia. Reduz o número de *contigs* falsos. | Requer alta cobertura (\(> 50x\)). *Reads* curtos que não atingem o comprimento \(k\) são descartados. |

> **Nota Metodológica**: O SPAdes executa um pipeline iterativo: ele constrói grafos com \(k\) pequenos para corrigir erros de sequenciamento (baseado em um modelo probabilístico de cobertura) e, em seguida, aumenta \(k\) progressivamente para resolver repetições complexas, fundindo os grafos em uma estrutura hierárquica final.

---

## 2.1.1 Execução Prática com SPAdes

O **SPAdes** (*St. Petersburg Genome Assembler*) é o *gold standard* para montagem de genomas bacterianos e arqueais. Seu algoritmo incorpora correção de erros Bayesiana, detecção de *reads* com cobertura anômala (contaminação) e módulo para plasmídeos.

### Comando Base

```bash
# Instalação via pacote do sistema (ou mamba/conda)
sudo apt install -y spades

# Criação do diretório de saída
mkdir -p assembly

# Execução do montador em modo rigoroso
spades.py --careful -o assembly/SRR10461876_assembly \
-1 trimmed_reads/SRR10461876_1_paired.fastq \
-2 trimmed_reads/SRR10461876_2_paired.fastq \
--cov-cutoff auto
