# Módulo 05: Introdução ao Linux para Bioinformática

A fluência no ambiente Linux é indispensável para o bioinformata moderno. Diferentemente da biologia experimental, onde o fluxo de trabalho é frequentemente guiado por interfaces gráficas (GUI), a bioinformática computacional opera predominantemente em ambientes de linha de comando (CLI), por três razões fundamentais: (i) **escalabilidade** – scripts processam centenas de amostras de forma padronizada; (ii) **reprodutibilidade** – cada comando executado fica registrado no histórico; e (iii) **acesso a recursos computacionais** – servidores de alto desempenho (HPC) e nuvens computacionais são acessados exclusivamente via terminal.

Este módulo não apenas ensina comandos, mas desenvolve a **mentalidade computacional** necessária para projetar *pipelines* robustos, diagnosticar falhas e otimizar o uso de recursos computacionais – habilidades essenciais para a genômica de microrganismos em grande escala.

---

## 0.1 Por que Linux na Bioinformática? Uma Visão Estratégica

| Dimensão | Benefício para a Genômica Computacional |
|:---|:---|
| **Economia de Recursos** | Servidores Linux utilizam recursos de forma mais eficiente que sistemas GUI, maximizando a memória RAM e CPU para montagem de genomas. |
| **Automação** | Scripts Bash permitem repetir análises completas com um único comando, reduzindo erros manuais. |
| **Padrão Industrial** | Praticamente todas as ferramentas de bioinfo (SPAdes, Prokka, BUSCO, Snakemake, Nextflow) são desenvolvidas primariamente para Linux. |
| **Acesso a HPC** | Clusters acadêmicos e nuvens (AWS, Google Cloud) utilizam Linux, permitindo processamento paralelo massivo. |
| **Controle de Versão** | Integração nativa com Git, facilitando o rastreamento de alterações em scripts e dados. |

> **Referência Fundamental**: O livro *Bioinformatics Data Skills* (Vince Buffalo) estabelece que a produtividade em bioinformática é diretamente proporcional à proficiência no terminal – *"aprender Linux é o investimento de maior retorno para qualquer biólogo computacional"*.

---

## 0.2 O Terminal, o Shell e o Ambiente de Execução

### O Terminal vs. o Shell
- **Terminal** (ou emulador): a janela gráfica que permite interagir com o shell.
- **Shell**: o interpretador de comandos. O **Bash** (*Bourne Again SHell*) é o mais comum, mas alternativas como **Zsh** também são populares.

### Prompt de Comando
```bash
usuario@bioinfra:~$
```
- **`usuario`**: identificador da sessão.
- **`bioinfra`**: nome da máquina (pode ser um servidor remoto).
- **`~`**: atalho para o diretório *home* do usuário (`/home/usuario`).
- **`$`**: indica usuário comum (o `#` seria o *root* – superusuário).

### Sessões Interativas vs. Não-Interativas
- **Interativa**: você digita comandos manualmente (uso em desenvolvimento e testes).
- **Não-Interativa**: scripts executam comandos automaticamente (uso em produção e *pipelines*).

---

## 0.3 Sistema de Arquivos e Organização de Dados Genômicos

### Estrutura Padrão para Projetos de Genômica (Recomendação)

```bash
projeto_genoma/
├── raw_data/          # FASTQ brutos (não modificar!)
├── logs/              # Arquivos de log e histórico
├── scripts/           # Scripts Bash e SLURM
├── trimmed_data/      # FASTQ trimados
├── assembly/          # Montagem (contigs, scaffolds)
├── annotation/        # Anotação (GFF, GBK, proteínas)
├── qc_reports/        # Relatórios FastQC, MultiQC, BUSCO
└── README.md          # Documentação do projeto
```

### Diretórios Especiais em HPC (Clusters)
Em servidores de alto desempenho, você encontrará diretórios específicos:
- **`/home/`** → capacidade limitada (ideal para scripts e configurações, NÃO para dados grandes).
- **`/scratch/`** ou **`/tmp/`** → espaço temporário de alto desempenho para arquivos intermediários (montagens, BAMs). Dados aqui são frequentemente apagados automaticamente após 30 dias.
- **`/work/`** ou **`/project/`** → espaço de armazenamento de médio/longo prazo para dados de projeto.

> **Boas práticas**: Mantenha dados brutos (`raw_data`) em um local estável e executável apenas a partir do `scratch` para maximizar a velocidade de I/O.

### Caminhos Absolutos e Relativos com Simbolismo
- **`~`** (til) representa o home.
- **`.`** diretório atual.
- **`..`** diretório pai.
- **`-`** diretório anterior (útil para voltar rapidamente: `cd -`).

---

## 0.4 Comandos Essenciais para Navegação e Manipulação de Arquivos

### Navegação e Listagem
```bash
pwd                        # Mostra o diretório atual
ls -l                      # Lista detalhada (permissões, tamanho, data)
ls -la                     # Inclui arquivos ocultos (.*)
ls -lh                     # Tamanhos legíveis (KB, MB, GB)
ls -lt                     # Ordena por data de modificação (mais recente primeiro)
du -sh *                   # Mostra o tamanho de cada diretório/arquivo (útil para verificar dados)
df -h                      # Verifica espaço disponível em disco
```

### Criação, Cópia, Movimento e Remoção
```bash
mkdir -p projeto/{raw_data,assembly,annotation}   # Cria subdiretórios aninhados
cp -r projeto/ backup_projeto/                     # Cópia recursiva (cuidado com arquivos grandes!)
mv arquivo_1.fastq arquivo_2.fastq data/           # Move e renomeia
rm -i arquivo.txt                                   # Remove com confirmação interativa (seguro!)
rm -rf temp_dir/                                    # Remove diretório FORÇADAMENTE (PERIGO!)
```

### Links Simbólicos (Atalhos Inteligentes)
Links simbólicos são cruciais para evitar duplicação de dados:
```bash
ln -s /scratch/projeto/raw_data ~/raw_data_link    # Cria um atalho para dados pesados
```

---

## 0.5 Visualização e Filtragem de Conteúdo com Ferramentas de Texto

### Visualização de Arquivos
```bash
cat arquivo.fasta                                    # Exibe todo o conteúdo (apenas para arquivos pequenos)
less -S arquivo.gff                                  # Navegação interativa (-S permite scroll horizontal)
head -n 4 SRR10461876_1.fastq                       # Exibe as 4 primeiras linhas (um read completo)
tail -f log_assembly.txt                             # Acompanha atualizações em tempo real
```

### Filtragem Avançada com `grep` (Busca de Padrões)
`grep` é a ferramenta mais utilizada para vasculhar arquivos de sequências:
```bash
grep "^>" genome.fasta                               # Lista todos os cabeçalhos de sequências FASTA
grep -c "^@" sample.fastq                           # Conta o número de reads no FASTQ
grep -v "^#" annotation.gff                         # Remove linhas de comentário
grep -E "gene|CDS" annotation.gff                   # Busca múltiplos padrões (expressões regulares estendidas)
zgrep "ACGT" sequences.fastq.gz                     # Busca dentro de arquivos compactados (sem descompactar!)
```

### Manipulação de Colunas com `cut` e `awk`
Em bioinformática, é comum trabalhar com arquivos tabulares (TSV, GFF, VCF):
```bash
# Extrair a primeira e terceira coluna de um GFF (seqid e tipo)
cut -f 1,3 annotation.gff | head

# Usando awk para cálculos e condicionais (ex: contar CDS em um GFF)
awk '$3 == "CDS"' annotation.gff | wc -l

# Awk para extrair o comprimento de sequências em um FASTA (identificação de > 1.000 pb)
awk '!/^>/ {print length($0)}' genome.fasta | sort -nr | head -10

# Awk para calcular o N50 (métrica de montagem) – algoritmo clássico
awk '!/^>/ {len[++i]=length($0)} END { 
    total=0; for (l in len) total+=len[l]; half=total/2; 
    asort(len); sum=0; for (j=length(len); j>=1; j--) { 
        sum+=len[j]; if (sum >= half) {print "N50 =", len[j]; break} 
    }
}' contigs.fasta
```

### Transformação com `sed` (Edição de Fluxo)
`sed` é um editor de texto não interativo que permite substituições e transformações em massa:
```bash
# Substituir cabeçalhos de um FASTA adicionando "sample1_" ao nome
sed 's/^>/>sample1_/' original.fasta > renamed.fasta

# Remover caracteres de retorno de carro (Windows -> Linux)
sed -i 's/\r$//' script.sh

# Extrair a região entre as coordenadas 100 e 200 em um FASTA (usando sed combinado com range)
sed -n '100,200p' large_file.txt
```

---

## 0.6 Trabalhando com Dados Compactados (*gzip*, *.gz*)

No sequenciamento de nova geração, arquivos FASTQ são massivos. O uso de compactação (`.gz`) é obrigatório. As ferramentas **não descompactam** para processamento, economizando tempo e espaço.

| Comando | Função | Exemplo Bioinfo |
|:---|:---|:---|
| `zcat` | Descompacta para a saída padrão | `zcat sample.fastq.gz \| head -n 4` |
| `zgrep` | Busca dentro do `.gz` | `zgrep -c "@SRR" sample.fastq.gz` |
| `zless` | Visualiza interativamente | `zless assembly.fasta.gz` |
| `gzip -d` | Descompacta (ou `gunzip`) | `gunzip file.fastq.gz` |
| `gzip` | Compacta | `gzip large_file.txt` |
| `pigz` | Compactação paralela (mais rápida em múltiplos núcleos) | `pigz -p 8 large_file.fastq` |

> **Prática padrão**: Mantenha os dados brutos sempre compactados para economizar espaço. Processe diretamente com ferramentas como `fastp` e `SPAdes`, que aceitam entrada `.gz` nativamente.

---

## 0.7 Redirecionamento, Pipes e Substituição de Comandos

### Pipes Múltiplos (Construindo um Pipeline de QC)
```bash
# Pipeline completo de controle de qualidade sem arquivos intermediários
zgrep -c "^@" sample_1.fastq.gz > total_reads.txt
zcat sample_1.fastq.gz | head -100000 | grep "^@" | wc -l  # Amostragem
zcat sample_1.fastq.gz | awk '{if(NR%4==2) print length($0)}' | sort -nr | head -20 # 20 maiores reads
```

### Substituição de Comandos (`$()`) – Aninhamento Inteligente
```bash
# Definir uma variável com o caminho de um arquivo encontrado dinamicamente
DATA_DIR=$(find /scratch -name "*_1.fastq.gz" | head -1)
echo "Processando $DATA_DIR"
```

### Processo de Substituição de Comandos com Pipes
```bash
# Calcular quantas vezes um adaptador específico aparece em todos os FASTQs
for f in *.fastq.gz; do echo $f; zgrep "GATC" $f | wc -l; done | paste - -
```

---

## 0.8 Permissões de Arquivo e Propriedade em Ambiente Colaborativo

Em clusters compartilhados, o controle de permissões é essencial para evitar a exclusão ou sobreposição de dados por outros usuários.

### Compreendendo a Saída de `ls -l`
`-rw-r--r-- 1 usuario grupo 1024 Jan 01 12:00 arquivo.txt`

| Posição | Interpretação |
|:---|:---|
| `-` | Arquivo (`d` para diretório) |
| `rw-` | Usuário (dono) tem leitura e escrita |
| `r--` | Grupo tem leitura |
| `r--` | Outros têm leitura |

### Modificando com `chmod` (Numérico e Simbólico)
```bash
chmod 755 script.sh        # rwxr-xr-x (dono escreve, grupo e outros executam)
chmod 644 data.txt         # rw-r--r-- (dono escreve, outros só leem)
chmod g+w annotation/      # Adiciona permissão de escrita para o grupo no diretório
chmod -R 750 projetos/     # Recursivo (aplica a todos os subdiretórios)
```

> **Recomendação para bioinfo**: Mantenha dados brutos como `644` (apenas leitura) e scripts como `755` (executáveis). Crie grupos de colaboradores (`usermod -aG nome_do_grupo usuario`) para compartilhar arquivos sem abrir permissões para todos.

---

## 0.9 Gerenciamento de Software Avançado: Conda/Mamba e Módulos HPC

### Estratégia Conda vs. Módulos de Ambiente
| Abordagem | Vantagem | Desvantagem |
|:---|:---|:---|
| **Conda/Mamba** | Gerencia dependências automaticamente; isola ambientes; portável | Pode ocupar muito espaço (múltiplas versões de bibliotecas) |
| **Module (LMOD)** | Padrão em HPC; carrega versões otimizadas para o cluster | Depende da instalação do administrador; não instala novas ferramentas facilmente |

### Comandos Conda/Mamba Avançados
```bash
# Criar um ambiente com versões específicas de Python e bibliotecas
mamba create -n assembly python=3.11 spades=3.15.5 quast=5.2.0

# Clonar um ambiente existente para testar alterações
mamba create --clone assembly --name assembly_test

# Exportar ambiente para reprodutibilidade (Criar um environment.yml)
mamba env export --no-builds > environment.yml
# Recriar exatamente o mesmo ambiente em outra máquina:
mamba env create -f environment.yml
```

### Usando Módulos em HPC (Exemplo: SLURM)
```bash
module avail                          # Lista os módulos disponíveis no sistema
module load gcc/11.2.0 bowtie2/2.4.5  # Carrega versões específicas
module list                           # Mostra os módulos carregados
module unload spades                  # Remove um módulo
```

---

## 0.10 Scripts Bash Robostos para Pipelines Genômicos

### Estrutura de um Script Profissional

```bash
#!/bin/bash
# ========================================================================
# Script: run_assembly.sh
# Descrição: Pipeline de montagem para genomas bacterianos (Illumina PE)
# Autor: Seu Nome
# Data: 2025-03-27
# Uso: ./run_assembly.sh -i raw_data -o assembly_output -t 8
# ========================================================================

# Configuração para parar o script se algum comando falhar (ESSENCIAL!)
set -euo pipefail  # -e: sai ao errar | -u: erro se variável não definida | -o pipefail: falha no pipe

# =========================================
# Função para exibir ajuda
# =========================================
usage() {
    echo "Uso: $0 [-i INPUT_DIR] [-o OUTPUT_DIR] [-t THREADS]"
    echo "  -i  Diretório com FASTQs trimados"
    echo "  -o  Diretório de saída para a montagem"
    echo "  -t  Número de threads (padrão: 4)"
    exit 1
}

# =========================================
# Parse de argumentos (moderno e robusto)
# =========================================
INPUT_DIR=""
OUTPUT_DIR=""
THREADS=4

while getopts "i:o:t:h" opt; do
    case "$opt" in
        i) INPUT_DIR="$OPTARG" ;;
        o) OUTPUT_DIR="$OPTARG" ;;
        t) THREADS="$OPTARG" ;;
        h) usage ;;
        *) usage ;;
    esac
done

# Validar argumentos obrigatórios
if [ -z "$INPUT_DIR" ] || [ -z "$OUTPUT_DIR" ]; then
    echo "Erro: Os parâmetros -i e -o são obrigatórios."
    usage
fi

# Verificar se o diretório de entrada existe
if [ ! -d "$INPUT_DIR" ]; then
    echo "Erro: Diretório $INPUT_DIR não encontrado."
    exit 1
fi

# Criar diretório de saída se não existir
mkdir -p "$OUTPUT_DIR"

# =========================================
# Configurar ambiente Conda
# =========================================
source /home/usuario/miniconda3/etc/profile.d/conda.sh
conda activate assembly_env

# =========================================
# Execução do Pipeline
# =========================================
echo "$(date) - Iniciando montagem com SPAdes..."

# Encontrar arquivos de entrada (wildcard robusto)
R1=$(ls $INPUT_DIR/*_1_paired.fastq.gz 2>/dev/null | head -1)
R2=$(ls $INPUT_DIR/*_2_paired.fastq.gz 2>/dev/null | head -1)

if [ -z "$R1" ] || [ -z "$R2" ]; then
    echo "Erro: Arquivos paired-end não encontrados em $INPUT_DIR"
    exit 1
fi

# Comando principal com logging
spades.py --careful \
    -o "$OUTPUT_DIR" \
    -1 "$R1" -2 "$R2" \
    --threads "$THREADS" \
    2>&1 | tee "$OUTPUT_DIR/spades.log"

# Verificar se a montagem gerou o arquivo esperado
if [ -f "$OUTPUT_DIR/contigs.fasta" ]; then
    echo "$(date) - Montagem concluída com sucesso!"
    echo "Contigs gerados: $(grep -c '^>' $OUTPUT_DIR/contigs.fasta)"
    echo "Tamanho total: $(awk '!/^>/ {sum+=length($0)} END {print sum}' $OUTPUT_DIR/contigs.fasta)"
else
    echo "ERRO CRÍTICO: SPAdes não gerou contigs.fasta. Verifique o log."
    exit 1
fi

conda deactivate
echo "$(date) - Pipeline finalizado."
```

---

## 0.11 Gerenciamento de Processos e Sessões Persistentes

### Controle de Jobs em Background
```bash
# Executar um processo em background (soltando o terminal)
nohup spades.py -1 R1.fastq -2 R2.fastq -o assembly/ > spades.out 2> spades.err &

# Visualizar processos em execução
jobs          # Lista processos em segundo plano na sessão atual
ps aux | grep spades  # Ver todos os processos no sistema
htop          # Monitor interativo de CPU/Memória (superior ao top)

# Matar um processo (com força se necessário)
kill -15 PID   # Terminação gentil (SIGTERM)
kill -9 PID    # Terminação forçada (SIGKILL) – use apenas como último recurso
```

### Sessões Persistentes: Screen vs. Tmux
Em clusters onde a conexão pode cair, use sessões persistentes:
```bash
# SCREEN
screen -S montagem          # Inicia nova sessão nomeada
Ctrl+A, D                   # Desanexa (detach) – deixa rodando em background
screen -ls                  # Lista sessões
screen -r montagem          # Reanexa à sessão

# TMUX (mais moderno)
tmux new -s montagem        # Inicia nova sessão
Ctrl+B, D                   # Desanexa
tmux ls                     # Lista
tmux attach -t montagem     # Reanexa
```

---

## 0.12 Computação Paralela com `xargs` e `GNU Parallel`

O processamento de múltiplas amostras é uma tarefa rotineira. Em vez de loops sequenciais, use paralelização para reduzir drasticamente o tempo.

### Usando `xargs` para Paralelizar
```bash
# Executar FastQC em todos os arquivos FASTQ usando 4 threads
ls *.fastq.gz | xargs -P 4 -I {} fastqc {} -o qc_results/

# Rodar Prokka para todos os genomas em paralelo
ls assembly/*.fasta | xargs -P 2 -I {} prokka --outdir annotation/{} --force {}
```

### GNU Parallel (Mais poderoso e legível)
```bash
# Instalar via Conda
mamba install -c conda-forge parallel

# Processar 100 amostras com distribuição automática de carga
parallel -j 4 'fastqc {} -o qc_output/' ::: *.fastq.gz

# Combinar com substituição de argumentos (ex: trimagem pareada)
parallel -j 4 'trimmomatic PE -threads 2 {1} {2} trim/{1} trim/{2}' ::: R1_list.txt ::: R2_list.txt
```

---

## 0.13 Submissão de Jobs em Clusters HPC (SLURM)

Em clusters acadêmicos, a execução interativa é restrita. Usa-se o **SLURM** (*Simple Linux Utility for Resource Management*) para submissão de *jobs*.

### Script de Submissão SLURM (`run_assembly.sbatch`)

```bash
#!/bin/bash
#SBATCH --job-name=genome_assembly
#SBATCH --output=logs/assembly_%j.out
#SBATCH --error=logs/assembly_%j.err
#SBATCH --time=12:00:00          # Tempo máximo: 12 horas
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=16       # 16 CPUs
#SBATCH --mem=64G                # 64 GB de RAM
#SBATCH --partition=biopartition # Partição específica

# Carregar módulos
module load gcc/11.2.0 spades/3.15.5

# Executar o pipeline
cd /home/usuario/projeto_genoma
spades.py --careful -1 raw_data/sample_1.fastq.gz -2 raw_data/sample_2.fastq.gz \
    -o assembly/ --threads $SLURM_CPUS_PER_TASK
```

### Submetendo ao Cluster
```bash
sbatch run_assembly.sbatch
squeue -u usuario          # Verifica status dos jobs
scancel JOB_ID             # Cancela um job
sacct                      # Histórico de jobs concluídos
```

---

## 0.14 Ferramentas de Diagnóstico e Monitoramento

| Ferramenta | Função | Exemplo |
|:---|:---|:---|
| `htop` | Monitor de CPU/memória interativo | `htop` |
| `df -h` | Espaço disponível em disco | `df -h /scratch` |
| `du -sh *` | Tamanho de arquivos/diretórios | `du -sh raw_data/` |
| `lsof` | Arquivos abertos por processos | `lsof \| grep user` |
| `time` | Mede tempo de execução de um comando | `time spades.py ...` |
| `watch` | Executa comando repetidamente | `watch -n 10 squeue` |

---

## 0.15 Expressões Regulares (Regex) para Bioinformática

As expressões regulares são a base da filtragem em textos biológicos.

| Padrão | Exemplo | Aplicação |
|:---|:---|:---|
| `^>` | `grep "^>"` | Cabeçalhos de FASTA |
| `@SRR` | `zgrep "^@SRR"` | Identificadores de reads Illumina |
| `[ATCG]` | `grep "^[ATCG]\+$"` | Validação de sequências nucleotídicas |
| `\|\|` | `awk '$3=="gene" \|\| $3=="CDS"'` | Múltiplas condições em GFF |
| `\t` | `cut -f 1,3` (tab como delimitador) | Processamento de TSV/GFF |

---

## 📌 Síntese e Checklist de Proficiência

Ao final deste módulo, você deve ser capaz de:

1.  **Navegar** pelo sistema de arquivos e localizar dados rapidamente.
2.  **Manipular** arquivos FASTQ, FASTA e GFF usando `cat`, `grep`, `sed` e `awk`.
3.  **Processar** arquivos compactados `.gz` sem descompactá-los desnecessariamente.
4.  **Construir pipelines** encadeando comandos com pipes (`|`) e redirecionamentos.
5.  **Gerenciar ambientes** com Conda/Mamba, criando ambientes isolados e exportando `environment.yml`.
6.  **Escrever scripts Bash** com tratamento de erros (`set -euo pipefail`) e argumentos (`getopts`).
7.  **Executar análises em servidores remotos** via SSH com sessões persistentes (`screen`/`tmux`).
8.  **Paralelizar** análises com `xargs` ou `GNU parallel`.
9.  **Submeter jobs** em clusters SLURM com gerenciamento de recursos.
10.  **Monitorar recursos** (`htop`, `df`, `du`) para evitar estouro de memória/disco.

---

**Referências e Leituras Complementares**

- Buffalo, V. (2015). *Bioinformatics Data Skills: Reproducible and Robust Research with Open Source Tools*. O'Reilly Media.
- Shotts, W. (2019). *The Linux Command Line, 2nd Edition*. No Starch Press.
- Documentação Oficial SLURM: [slurm.schedmd.com](https://slurm.schedmd.com/)
- Documentação GNU Parallel: [gnu.org/software/parallel](https://www.gnu.org/software/parallel/)
- *Data Carpentry – Genomics Workshop*: [datacarpentry.org/genomics-workshop/](https://datacarpentry.org/genomics-workshop/)
