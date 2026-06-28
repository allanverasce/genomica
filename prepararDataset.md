# Preparação dos Dados para o Curso de Genômica

Para acompanhar as aulas práticas, você precisará de dois tipos de conjuntos de dados:

1. **Dados simulados (sintéticos)**: arquivos pequenos e rápidos de gerar, criados especificamente para treinar comandos de manipulação de texto (`grep`, `awk`, `sed`, `cut`, `cat`) e compreender a estrutura dos arquivos FASTA, FASTQ e GFF — sem a necessidade de baixar gigabytes de dados.

2. **Dados reais (sequenciamento Illumina)**: provenientes do repositório público NCBI SRA (acesso **SRR10461876**), correspondentes ao genoma de *Escherichia coli*. Estes serão utilizados nos pipelines completos de **Controle de Qualidade**, **Montagem** e **Anotação Genômica**.

---

## 1. Dados Simulados (Para treino no Módulo Linux)

Os dados simulados são gerados localmente com comandos nativos do Linux (não exigem instalação de ferramentas adicionais). Eles permitem praticar:

- Visualização e filtragem de arquivos FASTA/FASTQ.
- Busca de padrões com `grep` e expressões regulares.
- Manipulação de colunas em GFF com `cut` e `awk`.
- Redirecionamento e pipes.

### 1.1. Script automatizado (`generate_test_data.sh`)

Crie um arquivo chamado `generate_test_data.sh` no diretório raiz do curso e cole o conteúdo abaixo:

```bash
#!/bin/bash
# ============================================================================
# generate_test_data.sh – Gera dados simulados para o Módulo de Linux
# Uso: bash generate_test_data.sh
# ============================================================================

set -e  # Para o script se algum comando falhar

echo " Criando estrutura de diretórios..."
mkdir -p raw_data trimmed_reads assembly annotation qc_reports scripts

echo " Gerando sample.fasta (10 sequências de DNA)..."
cat > sample.fasta << 'EOF'
>seq1
ATCGATCGATCGATCGATCGATCGATCGATCGATCGATCG
>seq2
GCATGCATGCATGCATGCATGCATGCATGCATGCATGCAT
>seq3
TTTTCCCCAAAAGGGGTTTTCCCCAAAAGGGGTTTTCCCC
>seq4
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
>seq5
GATCAGATCAGATCAGATCAGATCAGATCAGATCAGATCA
>seq6
CGATCGATCGATCGATCGATCGATCGATCGATCGATCGAT
>seq7
NATCGATCGTAGCTAGCTAGCTAGCTAGCTAGCTAGCTAG
>seq8
CCCCGGGGAAAATTTTCCCCGGGGAAAATTTTCCCCGGGG
>seq9
AGCTAGCTAGCTAGCTAGCTAGCTAGCTAGCTAGCTAGC
>seq10
TATATATATATATATATATATATATATATATATATATATA
EOF

echo " Gerando sample.fastq (4 reads com qualidades codificadas)..."
cat > sample.fastq << 'EOF'
@read1
ATCGTACGATCGATCGATCGATCGATCGATCGATCGATCG
+
IIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIII
@read2
GCATGCATGCATGCATGCATGCATGCATGCATGCATGCAT
+
HHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHH
@read3
TTTTCCCCAAAAGGGGTTTTCCCCAAAAGGGGTTTTCCCC
+
GGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGG
@read4
GATCAGATCAGATCAGATCAGATCAGATCAGATCAGATCA
+
FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF
EOF

echo " Gerando annotation.gff (arquivo de anotação com CDS, rRNA, tRNA)..."
cat > annotation.gff << 'EOF'
##gff-version 3
contig_1	Prodigal	CDS	105	978	.	+	0	ID=cds00001;product=hypothetical protein
contig_1	Prodigal	CDS	1200	2100	.	-	0	ID=cds00002;product=DNA polymerase III subunit beta
contig_2	Barrnap	rRNA	1	1500	.	+	.	ID=rRNA00001;product=16S ribosomal RNA
contig_2	Prodigal	CDS	2000	3000	.	+	0	ID=cds00003;product=transposase
contig_3	tRNAscan-SE	tRNA	100	180	.	+	.	ID=tRNA00001;product=tRNA-Leu
contig_4	Prodigal	CDS	50	950	.	+	0	ID=cds00004;product=ABC transporter ATP-binding protein
contig_4	Prodigal	CDS	1200	2000	.	-	0	ID=cds00005;product=hypothetical protein
contig_5	Barrnap	rRNA	1	1200	.	+	.	ID=rRNA00002;product=23S ribosomal RNA
EOF

echo " Gerando reads trimados simulados (paired-end)..."
cat > trimmed_reads/SRR10461876_1_paired.fastq << 'EOF'
@read1
ATCGTACGATCGATCGATCGATCGATCGATCGATCGATCG
+
IIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIII
@read2
GCATGCATGCATGCATGCATGCATGCATGCATGCATGCAT
+
HHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHHH
EOF

cat > trimmed_reads/SRR10461876_2_paired.fastq << 'EOF'
@read1
GATCAGATCAGATCAGATCAGATCAGATCAGATCAGATCA
+
FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF
@read2
TTTTCCCCAAAAGGGGTTTTCCCCAAAAGGGGTTTTCCCC
+
GGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGG
EOF

echo "Dados simulados gerados com sucesso!"
echo "   - sample.fasta       (10 sequências)"
echo "   - sample.fastq       (4 reads)"
echo "   - annotation.gff     (9 anotações)"
echo "   - trimmed_reads/     (2 arquivos paired-end simulados)"
```

### 1.2. Executando o script

```bash
# Tornar o script executável
chmod +x generate_test_data.sh

# Executar
./generate_test_data.sh
```

### 1.3. Verificando os arquivos gerados

```bash
# Listar os arquivos criados
ls -lh sample.* annotation.gff
ls -lh trimmed_reads/

# Visualizar os primeiros reads do FASTQ simulado
head -n 8 sample.fastq
```

---

## 2. Dados Reais (Para os pipelines de QC, Montagem e Anotação)

Os dados reais utilizados no curso são do acesso **SRR10461876**, correspondente a um sequenciamento Illumina paired‑end do genoma de *Escherichia coli* (tamanho aproximado de 4,6 Mb). Eles serão baixados diretamente do Sequence Read Archive (SRA) do NCBI.

### 2.1. Instalando o SRA Toolkit

O **SRA Toolkit** é necessário para converter os dados do formato binário `.sra` para o formato textual FASTQ.

```bash
# Baixar a versão mais recente para Linux (64 bits)
wget https://ftp.ncbi.nlm.nih.gov/sra/sdk/current/sratoolkit.current-ubuntu64.tar.gz

# Descompactar
tar -xzf sratoolkit.current-ubuntu64.tar.gz

# Adicionar ao PATH (substitua pela versão baixada)
export PATH=$(pwd)/sratoolkit.3.2.1-ubuntu64/bin:$PATH

# Testar a instalação
fastq-dump --version
```

> 💡 **Dica**: Para tornar o `fastq-dump` disponível em todas as sessões do terminal, adicione a linha `export PATH=$PATH:/caminho/para/sratoolkit/bin` ao arquivo `~/.bashrc`.

### 2.2. Baixando os dados

```bash
# Criar diretório para os dados brutos
mkdir -p raw_data
cd raw_data

# Baixar os reads paired-end com compactação integrada
fastq-dump --split-files --gzip SRR10461876

# Voltar ao diretório principal
cd ..
```

**Explicação dos parâmetros**:
| Parâmetro | Função |
|:---|:---|
| `--split-files` | Separa os reads forward e reverse em dois arquivos (obrigatório para dados paired‑end). |
| `--gzip` | Compacta os arquivos em `.gz` para economizar espaço (~70% de redução). |

**Arquivos gerados**:
- `raw_data/SRR10461876_1.fastq.gz` → Reads forward (fita 1).
- `raw_data/SRR10461876_2.fastq.gz` → Reads reverse (fita 2).

### 2.3. Verificando a integridade dos dados

```bash
# Tamanho dos arquivos (esperado: ~200–300 MB cada)
du -sh raw_data/*.fastq.gz

# Visualizar as primeiras linhas de um arquivo compactado
zcat raw_data/SRR10461876_1.fastq.gz | head -n 8

# Contar o número de reads (cada read ocupa 4 linhas)
zcat raw_data/SRR10461876_1.fastq.gz | wc -l | awk '{print $1/4}'
```

---

## 3. Estrutura Final dos Diretórios

Após a geração dos dados simulados e reais, sua árvore de diretórios deve ficar assim:

```
curso_genomica/
├── generate_test_data.sh          # Script para dados simulados
├── sample.fasta                    # FASTA simulado
├── sample.fastq                    # FASTQ simulado
├── annotation.gff                  # GFF simulado
├── raw_data/                       # Dados reais
│   ├── SRR10461876_1.fastq.gz
│   └── SRR10461876_2.fastq.gz
├── trimmed_reads/                  # Dados trimados (simulados + futuros reais)
│   ├── SRR10461876_1_paired.fastq
│   └── SRR10461876_2_paired.fastq
├── assembly/                       # Resultados da montagem
├── annotation/                     # Resultados da anotação
├── qc_reports/                     # Relatórios de qualidade
└── scripts/                        # Scripts Bash/SLURM
```

---

## 4. Comandos Úteis para Explorar os Dados

Aproveite os arquivos gerados para praticar os comandos do Módulo Linux:

```bash
# Contar sequências no FASTA
grep -c "^>" sample.fasta

# Extrair apenas o cabeçalho do FASTA
grep "^>" sample.fasta

# Calcular o comprimento médio das sequências no FASTA
awk '!/^>/ {sum+=length($0); count++} END {print sum/count}' sample.fasta

# Buscar genes do tipo CDS no GFF
grep -P "\tCDS\t" annotation.gff

# Extrair a coluna de tipo (3ª coluna) do GFF e ordenar
cut -f 3 annotation.gff | sort | uniq -c

# Filtrar reads do FASTQ com cabeçalhos começando com "read"
grep "^@read" sample.fastq

# Contar reads no FASTQ real (compactado)
zgrep -c "^@" raw_data/SRR10461876_1.fastq.gz
```

---

## 5. Resumo

| Tipo de dado | Finalidade | Tamanho | Método de obtenção |
|:---|:---|:---|:---|
| **Simulados** (FASTA, FASTQ, GFF) | Treino de comandos, expressões regulares, scripts | Poucos KB | Script `generate_test_data.sh` |
| **Reais** (SRR10461876) | Pipeline completo de montagem e anotação | ~200–300 MB cada | SRA Toolkit + `fastq-dump` |

Agora você está pronto para iniciar o Módulo 0 com todos os dados necessários! 🚀
```
