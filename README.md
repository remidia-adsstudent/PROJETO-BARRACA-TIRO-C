# PROJETO-BARRACA-TIRO-C

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>

#define LINHAS 5
#define COLUNAS 5
#define QTD_PREMIOS 6

// --- STRUCTS E STRUCTS ANINHADAS ---

typedef struct {
int linha;
int coluna;
} Coordenada;

typedef struct {
int rolhas;
int pontos;
} Status;

typedef struct {
char nome[40];
int valor;
int derrubado;
Coordenada pos; // Struct aninhada
} Premio;

typedef struct {
char nome[40];
Status stats; // Struct aninhada
Coordenada mira; // Struct aninhada
} Jogador;

// --- PROTÓTIPOS DAS FUNÇÕES ---
void prepararArquivoHistoria();
void exibirHistoria();
void inicializarPainel(char painel[LINHAS][COLUNAS]);
void sortearPremios(Premio premios[], int qtd, char painel[LINHAS][COLUNAS]);
void exibirPainel(char painel[LINHAS][COLUNAS], Jogador j);
int processarTiro(Jogador *j, Premio premios[], int qtd, char painel[LINHAS][COLUNAS]);
void salvarRanking(Jogador j);
void exibirRanking();

// --- FUNÇÃO PRINCIPAL ---
int main() {
srand(time(NULL));

char painel[LINHAS][COLUNAS];
Premio premios[QTD_PREMIOS] = {
    {"Rapadura da Boa", 15, 0, {0, 0}},
    {"Chapeu de Couro", 30, 0, {0, 0}},
    {"Mini Sanfona", 50, 0, {0, 0}},
    {"Pacoca Doce", 10, 0, {0, 0}},
    {"Prato de Cuscuz", 25, 0, {0, 0}},
    {"Balao de Sao Joao", 20, 0, {0, 0}}
};

Jogador jogador;
jogador.stats.rolhas = 8;
jogador.stats.pontos = 0;

// 1. Manipulação de arquivo (Cria se não existir e lê a história)
prepararArquivoHistoria();
exibirHistoria();

printf("Digite seu nome, cabra da peste: ");
fgets(jogador.nome, sizeof(jogador.nome), stdin);
jogador.nome[strcspn(jogador.nome, "\n")] = '\0'; // Remove o '\n'

// 2. Inicializa matriz e distribui os structs de prêmios
inicializarPainel(painel);
sortearPremios(premios, QTD_PREMIOS, painel);

int premiosRestantes = QTD_PREMIOS;

// 3. Loop principal do jogo
while (jogador.stats.rolhas > 0 && premiosRestantes > 0) {
    exibirPainel(painel, jogador);

    printf("\nEscolha a LINHA para mirar (0 a %d): ", LINHAS - 1);
    scanf("%d", &jogador.mira.linha);
    printf("Escolha a COLUNA para mirar (0 a %d): ", COLUNAS - 1);
    scanf("%d", &jogador.mira.coluna);

    // Validação de coordenadas
    if (jogador.mira.linha < 0 || jogador.mira.linha >= LINHAS ||
        jogador.mira.coluna < 0 || jogador.mira.coluna >= COLUNAS) {
        printf("\n>>> Oxente! Voce atirou pra fora da barraca! Tente de novo. <<<\n");
        continue;
    }

    // Verifica se já atirou nesse local
    if (painel[jogador.mira.linha][jogador.mira.coluna] == 'X' ||
        painel[jogador.mira.linha][jogador.mira.coluna] == '*') {
        printf("\n>>> Ei, visse! Voce ja atirou nesse lugar! Escolha outro. <<<\n");
        continue;
    }

    // Processa o tiro usando função com retorno
    int acertou = processarTiro(&jogador, premios, QTD_PREMIOS, painel);
    if (acertou) {
        premiosRestantes--;
    }
}

// 4. Fim de jogo
printf("\n====================================================\n");
if (premiosRestantes == 0) {
    printf("ARRETADO! Voce limpou a barraca inteira, %s!\n", jogador.nome);
} else {
    printf("Acabaram suas rolhas, %s! Mais sorte no proximo Sao Joao.\n", jogador.nome);
}
printf("Pontuacao Final: %d pontos\n", jogador.stats.pontos);
printf("====================================================\n");

// 5. Persistência: Salva e mostra o ranking
salvarRanking(jogador);
exibirRanking();

return 0;
}

// --- IMPLEMENTAÇÃO DAS FUNÇÕES ---

// Garante que o arquivo historia.txt exista antes de tentar ler
void prepararArquivoHistoria() {
FILE *arq = fopen("historia.txt", "r");
if (arq == NULL) {
arq = fopen("historia.txt", "w");
if (arq != NULL) {
fprintf(arq, "====================================================\n");
fprintf(arq, " BARRACA DO TIRO AO ALVO - ARRAIAL DO NORDESTE \n");
fprintf(arq, "====================================================\n");
fprintf(arq, "Bem-vindo ao maior Sao Joao do mundo! O sanfoneiro ja\n");
fprintf(arq, "puxou o fole e a barraca de tiro ao alvo esta cheia.\n");
fprintf(arq, "Existem 6 premios escondidos no painel (5x5).\n");
fprintf(arq, "Use suas 8 rolhas com sabedoria para derrubar todos!\n");
fprintf(arq, "----------------------------------------------------\n\n");
fclose(arq);
}
} else {
fclose(arq);
}
}

// Lê a história do arquivo .txt usando fgets
void exibirHistoria() {
FILE *arq = fopen("historia.txt", "r");
char linha[120];

if (arq == NULL) {
    printf("Erro ao abrir historia.txt!\n");
    return;
}

while (fgets(linha, sizeof(linha), arq) != NULL) {
    printf("%s", linha);
}
fclose(arq);
}

// Preenche a matriz com o caractere '~' (painel intacto)
void inicializarPainel(char painel[LINHAS][COLUNAS]) {
for (int i = 0; i < LINHAS; i++) {
for (int j = 0; j < COLUNAS; j++) {
painel[i][j] = '~';
}
}
}

// Sorteia posições únicas na matriz para cada struct Premio
void sortearPremios(Premio premios[], int qtd, char painel[LINHAS][COLUNAS]) {
int ocupado[LINHAS][COLUNAS] = {0};

for (int i = 0; i < qtd; i++) {
    int l, c;
    do {
        l = rand() % LINHAS;
        c = rand() % COLUNAS;
    } while (ocupado[l][c] == 1);

    ocupado[l][c] = 1;
    premios[i].pos.linha = l;
    premios[i].pos.coluna = c;
}
}

// Imprime a matriz e os status do jogador
void exibirPainel(char painel[LINHAS][COLUNAS], Jogador j) {
printf("\n--- PAINEL DA BARRACA ---\n");
printf("Rolhas restantes: %d | Pontos: %d\n\n", j.stats.rolhas, j.stats.pontos);

printf("    ");
for (int c = 0; c < COLUNAS; c++) {
    printf(" %d ", c);
}
printf("\n   +---------------+\n");

for (int l = 0; l < LINHAS; l++) {
    printf(" %d |", l);
    for (int c = 0; c < COLUNAS; c++) {
        printf(" %c ", painel[l][c]);
    }
    printf("|\n");
}
printf("   +---------------+\n");
printf("Legenda: [~] Alvo oculto | [*] Premio acertado | [X] Tiro na agua\n");
}

// Verifica se a coordenada do tiro acertou algum prêmio (Função com retorno)
int processarTiro(Jogador *j, Premio premios[], int qtd, char painel[LINHAS][COLUNAS]) {
j->stats.rolhas--;

for (int i = 0; i < qtd; i++) {
    if (!premios[i].derrubado &&
        premios[i].pos.linha == j->mira.linha &&
        premios[i].pos.coluna == j->mira.coluna) {

        premios[i].derrubado = 1;
        j->stats.pontos += premios[i].valor;
        painel[j->mira.linha][j->mira.coluna] = '*';

        printf("\n*** EITA PESTE! ACERTOU NA MOSCA! ***\n");
        printf("Voce derrubou: %s (+%d pontos)!\n", premios[i].nome, premios[i].valor);
        return 1; // Retorna 1 indicando acerto
    }
}

painel[j->mira.linha][j->mira.coluna] = 'X';
printf("\n>>> Passou longe, cumpadi! Voce acertou so a madeira. <<<\n");
return 0; // Retorna 0 indicando erro
}

// Salva a pontuação do jogador no arquivo ranking.txt usando fprintf
void salvarRanking(Jogador j) {
FILE *arq = fopen("ranking.txt", "a");
if (arq == NULL) {
printf("Nao foi possivel salvar o ranking.\n");
return;
}

fprintf(arq, "Jogador: %-20s | Pontuacao: %d pts\n", j.nome, j.stats.pontos);
fclose(arq);
}

// Lê e exibe o ranking salvo
void exibirRanking() {
FILE *arq = fopen("ranking.txt", "r");
char linha[100];

printf("\n=== RANKING DO ARRAIAL (ranking.txt) ===\n");
if (arq == NULL) {
    printf("Nenhum registro encontrado.\n");
    return;
}

while (fgets(linha, sizeof(linha), arq) != NULL) {
    printf("%s", linha);
}
printf("========================================\n");
fclose(arq);
}

