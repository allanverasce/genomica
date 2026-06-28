# Módulo 2: Montagem do Genoma – Reconstrução e Validação Estrutural

A montagem *de novo* é o processo central da genômica computacional, no qual fragmentos curtos de DNA (*reads*) são reunidos para reconstruir a arquitetura original do genoma. Diferente do alinhamento contra uma referência, a montagem *de novo* não pressupõe conhecimento prévio da sequência-alvo, sendo essencial para organismos não-modelo, cepas emergentes ou estudos metagenômicos. Neste módulo, abordaremos a implementação prática com o montador **SPAdes**, a inspeção visual do grafo de montagem e a validação estatística por meio de métricas de contiguidade (QUAST) e completude funcional (BUSCO).

---

## Objetivo Geral do Processo de Montagem

O fluxo abaixo sintetiza as etapas críticas: desde a correção de erros nos *reads* até a geração de *contigs* e *scaffolds*, passando pela resolução de regiões repetitivas e validação final da qualidade.

<img src="imgs/etapasM.png" alt="Etapas de Montagem" width="800" height="600" />
*Fluxograma representativo das etapas de montagem *de novo*: correção de erros → construção do grafo de Bruijn → resolução de repetições → geração de sequências contíguas (contigs/scaffolds) → avaliação da qualidade.*

---

## 2.1 Montagem de Genomas (*De Novo Assembly*)

### Fundamentação Teórica: O Grafo de Bruijn e os *k‑mers*

Montadores para dados de *short‑reads* (Illumina) utilizam o **grafo de Bruijn** (*de Bruijn graph*). Neste modelo:

1. Os *reads* são decompostos em todas as subsequências de comprimento fixo \(k\), denominadas ***k‑mers***.
2. Cada *k‑mer* único torna‑se um **nó** no grafo.
3. Dois nós são conectados por uma **aresta** direcionada se houver sobreposição de \((k-1)\) bases (ex: `ACG` → `CGT`).
4. O genoma reconstruído corresponde a um caminho (*path*) que percorre esses nós.

O SPAdes adota uma abordagem **multi‑k‑mer**, testando automaticamente múltiplos valores de \(k\) (ex: 21, 33, 55, 77, 99, 127) para equilibrar sensibilidade (resolver regiões de baixa cobertura) e especificidade (resolver repetições longas). O tamanho ideal de \(k\) é geralmente o maior valor possível sem exceder o comprimento dos *reads*.

<img src="imgs/grafo.png" alt="Grafos" width="800" height="600" />
*Representação esquemática de um grafo de Bruijn. Nós (vértices) são *k‑mers*; arestas indicam sobreposições. Caminhos lineares representam regiões não repetitivas; bifurcações (*bubbles*) indicam polimorfismos ou erros de sequenciamento.*

### Execução Prática com SPAdes

O **SPAdes** (*St. Petersburg Genome Assembler*) é o *gold standard* para montagem de genomas bacterianos e arqueais, com suporte a dados Illumina, Ion Torrent e híbridos (curtos + longos). Seu algoritmo incorpora correção de erros Bayesiana, detecção de cobertura anômala e módulo dedicado a plasmídeos.

**Comando base**:

```bash
# Instalação via gerenciador de pacotes do sistema
sudo apt install -y spades

# Criação do diretório de saída
mkdir assembly

# Execução principal
spades.py --careful -o assembly/SRR10461876_assembly \
-1 trimmed_reads/SRR10461876_1_paired.fastq \
-2 trimmed_reads/SRR10461876_2_paired.fastq \
--cov-cutoff auto
