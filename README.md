== Dungeon Crawler ==

Projeto desenvolvido em linguagem C para a disciplina de Algoritmos e codificações de sistemas, do professor Pedro Girotto, do curso de Ciência da Computação do Cesupa.

Integrantes

Arthur Picanço Thiago Sawada Arthur Moraes

Descrição

Dungeon Crawler é um jogo de exploração em terminal onde o jogador deve atravessar três andares de uma masmorra, coletando chaves, abrindo portas, evitando armadilhas e derrotando monstros até enfrentar o chefe final, um Cyclope.

História

Ha muitos anos, uma Dungeon surgiu nas proximidades do reino.

Criaturas perigosas passaram asair de suas profundezas e atacar os viajantes da regiao, diz a lenda que um terrivel Cyclope vive la.

Um aventureiro foi escolhido para entrar na Dungeon e derrotar acriatura responsavel por tudo isso. Sua jornada comeca agora...

Funçoes do jogo / Como jogar

'*' = Parede . = Chao '#' = Espinho k = Caixa @ = Chave D = Porta Fechada = = Porta Aberta L = Escada O = Botao N = NPC Z = Boss(Cyclope) X = Monstro X(Goblin) Y = Monstro Y(Esqueleto)

Controles

w = Cima a = Esquerda s = Baixo d = Direita o = Atacar i = Interagir Q = Sair

Objetivo

Matar o cyclope e salvar a vila e todos os seus residentes.

Declaraçao do uso de IA generativa

a IA chatgpt foi utilizada para os ensinamentos de typedef struct, alem de ajudar com os "for" do codigo, a mesma foi usada para ajudar a corrigir bugs do jogo e na criaçao da funcao void.

Codigo do jogo:
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

#define MAX 25	// Tamanho maximo possivel do mapa.

#define PAREDE '*'
#define CHAO '.'
#define ESPINHO '#'
#define CAIXA 'k'
#define PORTA 'D'			// Define os simbolos do jogo.
#define PORTA_ABERTA '='
#define CHAVE '@'
#define ESCADA 'L'
#define NPC 'N'
#define BOTAO 'O'

// Struct Do Jogador: Armazena todas as informacoes importantes do player.
typedef struct
{
int x;          // Posicao X do jogador.
int y;          // Posicao Y do jogador.
int vidas;      // Quantidade de vidas.
int chaves;     // Quantidade de chaves.
char dir;       // Direcao que o jogador esta olhando.
} Player;

// Struct Dos Monstros: Armazena todas as informacoes importantes dos monstros.
typedef struct
{
int x;
int y;
int vivo;
} Monstro;

// Struct Do Boss: Armazena todas as informacoes importantes do boss.
typedef struct
{
int x;
int y;
int vivo;
int vida;
int cooldownataque;
} Boss;

char mapa[MAX][MAX];	// Matriz principal do mapa.

// Variaveis para controlar o tamanho do mapa.
int larguramapa;
int alturamapa;

// Variavel para controlar a fase atual.
// 0 = Vila
// 1 = Fase 1
// 2 = Fase 2
// 3 = Fase 3
int faseatual = 0;

// Variavel global para o jogador.
Player player;

// Variavel global para os monstros.
Monstro monstroX;
Monstro monstroY;

// Variavel global para o boss
Boss boss;

// 0 = nenhuma
// 1 = espada
// 2 = arco
// 3 = cajado
int armaescolhida = 0;

//se o NPC falou com o jogador
int armaselecionada = 0;

int gameover = 0;
int botaoativado = 0;

// Prototipos das funções.
void carregarmapa();
void carregarvila();
void carregarfase1();
void carregarfase2();
void carregarfase3();

void desenharmapa();
void mover(char tecla);
void atacar();
void atacarespada();
void atacararco();
void atacarcajado();
void matarmonstro(int x, int y);
void movermonstros();
void movermonstroX();
void movermonstroY();
void moverboss();
void interagir();
void perdervida();

void jogo();
void tutorial();

int main()
{
srand(time(NULL));

int opcao;

// loop do menu.
while(1)
{
	system("cls || clear");

	printf("===== DUNGEON CRAWLER =====\n\n");

	printf("1 - Jogar\n");
	printf("2 - Tutorial\n");
	printf("3 - Sair\n\n");

	printf("Escolha uma opcao: ");
	scanf("%d", &opcao);

	// Switch para selecionar a opção do menu.
	switch(opcao)
	{
	case 1:
		jogo();
		break;

	case 2:
		tutorial();
		break;

	case 3:
		printf("\n===== Créditos =====\n");
		printf("Arthur Damaso De Andrade Picanco\n");
		printf("Thiago Sawada Kitajima\n");
		printf("Arthur Moraes De Souza\n");
		return 0;

	default:
		printf("\nOpcao invalida!\n");
		system("pause");
	}
}

return 0;
}

// Função principal do jogo.
void jogo()
{
char tecla;

system("cls || clear");

printf("=================================\n");
printf("          PROLOGO\n");
printf("=================================\n\n");

printf("Ha muitos anos, uma Dungeon surgiu\n");
printf("nas proximidades do reino.\n\n");

printf("Criaturas perigosas passaram a\n");
printf("sair de suas profundezas e atacar\n");
printf("os viajantes da regiao, diz a lenda que um terrivel Cyclope vive la.\n\n");

printf("Um aventureiro foi escolhido para\n");
printf("entrar na Dungeon e derrotar a\n");
printf("criatura responsavel por tudo isso.\n\n");

printf("Sua jornada comeca agora...\n\n");

system("pause");

// Posição inicial do jogador.
player.x = 1;
player.y = 1;

// Quantidade inicial de vidas.
player.vidas = 3;

// Volta para o meu após morrer.
gameover = 0;

// Quantidade inicial de chaves.
player.chaves = 0;

// Quantidade inicial de armas.
armaescolhida = 0;
armaselecionada = 0;

// Direção inicial do player.
player.dir = '>';

// Começa na vila.
faseatual = 0;

// Carrega o mapa da fase.
carregarmapa();

// Loop principal do jogo.
while(1)
{
	if(gameover)
	{
		return;
	}

	// Desenha o mapa na tela.
	desenharmapa();

	// Lê o comando do jogador.
	printf("\nComando: ");
	scanf(" %c", &tecla);

	// Sair do jogo.
	if(tecla == 'q' || tecla == 'Q')
		break;

	// Verifica o comando digitado.
	switch(tecla)
	{
		// Teclas De Movimento.
	case 'w':
	case 'W':
	case 'a':
	case 'A':
	case 's':
	case 'S':
	case 'd':
	case 'D':
		mover(tecla);
		break;

		// Ataque
	case 'o':
	case 'O':
		atacar();
		break;

		// Interação
	case 'i':
	case 'I':
		interagir();
		break;
	}

	movermonstros();
}
}

// Função para carregar a fase atual do jogo.
void carregarmapa()
{
switch(faseatual)
{
	// Vila
case 0:
	carregarvila();
	break;

	// Primeiro Andar
case 1:
	carregarfase1();
	break;

	// Segundo Andar
case 2:
	carregarfase2();
	break;

	// Terceiro Andar
case 3:
	carregarfase3();
	break;
}
}

// Função que carrega a vila.
void carregarvila()
{
int i, j;

// Tamanho do mapa da vila 10x10.
larguramapa = 10;
alturamapa = 10;

// Cria o mapa da vila.
for(i = 0; i < alturamapa; i++)
{
	for(j = 0; j < larguramapa; j++)
	{
		// Cria as paredes e a borda.
		if(i == 0 || j == 0 ||
				i == alturamapa - 1 ||
				j == larguramapa - 1)
		{
			mapa[i][j] = PAREDE;
		}
		else
		{
			// Preenche o resto com chao.
			mapa[i][j] = CHAO;
		}
	}
}

// Elementos da fase da vila.
mapa[4][4] = NPC;
mapa[8][8] = ESCADA;
}

// Primeira Fase.
void carregarfase1()
{
int i, j;

// Tamanho 10x10.
larguramapa = 10;
alturamapa = 10;

// Cria o mapa do primeiro andar.
for(i = 0; i < alturamapa; i++)
{
	for(j = 0; j < larguramapa; j++)
	{
		if(i == 0 || j == 0 ||
				i == alturamapa - 1 ||
				j == larguramapa - 1)
		{
			mapa[i][j] = PAREDE;
		}
		else
		{
			mapa[i][j] = CHAO;
		}
	}
}

// Elementos da fase do primeiro andar.

// Parede horizontal
mapa[3][1] = PAREDE;
mapa[3][2] = PAREDE;
mapa[3][3] = PAREDE;
mapa[3][4] = PAREDE;
mapa[3][6] = PAREDE;
mapa[3][7] = PAREDE;
mapa[3][8] = PAREDE;

// Itens
mapa[2][2] = CHAVE;
mapa[3][5] = PORTA;
mapa[7][7] = CAIXA;
mapa[8][7] = CAIXA;
mapa[7][8] = CAIXA;
mapa[8][8] = ESCADA;

}

// Segunda Fase.
void carregarfase2()
{
int i, j;

botaoativado = 0;

// Tamanho 15x15.
larguramapa = 15;
alturamapa = 15;

// Cria o mapa.
for(i = 0; i < alturamapa; i++)
{
	for(j = 0; j < larguramapa; j++)
	{
		if(i == 0 || j == 0 ||
				i == alturamapa - 1 ||
				j == larguramapa - 1)
		{
			mapa[i][j] = PAREDE;
		}
		else
		{
			mapa[i][j] = CHAO;
		}
	}
}

mapa[2][1] = PAREDE;
mapa[2][2] = PAREDE;
mapa[2][3] = PAREDE;
mapa[2][4] = PAREDE;
mapa[2][5] = PAREDE;
mapa[2][6] = PAREDE;
mapa[2][7] = PAREDE;
mapa[2][8] = PAREDE;
mapa[2][9] = PAREDE;
mapa[2][10] = PAREDE;
mapa[2][11] = PAREDE;
mapa[2][12] = PAREDE;
mapa[2][13] = PAREDE;
mapa[11][10] = PAREDE;
mapa[11][11] = PAREDE;
mapa[11][12] = PAREDE;
mapa[11][13] = PAREDE;
mapa[13][10] = PAREDE;

mapa[1][13] = BOTAO;

mapa[3][3] = ESPINHO;
mapa[3][4] = ESPINHO;
mapa[3][5] = ESPINHO;

mapa[5][1] = PAREDE;
mapa[5][2] = PAREDE;
mapa[5][3] = PAREDE;
mapa[5][4] = PAREDE;
mapa[5][5] = ESPINHO;
mapa[5][6] = ESPINHO;
mapa[5][7] = ESPINHO;
mapa[5][8] = PAREDE;
mapa[5][9] = PORTA;
mapa[5][10] = PAREDE;
mapa[5][11] = PAREDE;
mapa[5][12] = PAREDE;
mapa[5][13] = PAREDE;

mapa[4][11] = CHAVE;

mapa[11][4] = CHAVE;

mapa[12][10] = PORTA;

mapa[7][12] = CAIXA;
mapa[8][12] = CAIXA;
mapa[8][11] = CAIXA;
mapa[8][3] = CAIXA;
mapa[8][4] = CAIXA;
mapa[9][3] = CAIXA;

mapa[13][13] = ESCADA;

// Monstro X
monstroX.x = 8;
monstroX.y = 8;
monstroX.vivo = 1;

mapa[monstroX.y][monstroX.x] = 'X';
}

// Terceira Fase.
void carregarfase3()
{
int i, j;

// Tamanho 25x25.
larguramapa = 25;
alturamapa = 25;

// Cria o mapa do terceiro andar.
for(i = 0; i < alturamapa; i++)
{
	for(j = 0; j < larguramapa; j++)
	{
		if(i == 0 || j == 0 ||
				i == alturamapa - 1 ||
				j == larguramapa - 1)
		{
			mapa[i][j] = PAREDE;
		}
		else
		{
			mapa[i][j] = CHAO;
		}
	}
}

// Elementos da fase do terceiro andar.

//Paredes
int h;

for(h = 3; h <= 23; h++)
{
	mapa[4][h] = PAREDE;
}
for(h = 3; h <= 23; h++)
{
	mapa[10][h] = PAREDE;
}
for(h = 1; h <= 11; h++)
{
	mapa[17][h] = PAREDE;
}

for(h = 13; h <= 23; h++)
{
	mapa[17][h] = PAREDE;
}

mapa[4][1] = PAREDE;
mapa[10][1] = PAREDE;
mapa[5][15] = PAREDE;
mapa[6][6] = PAREDE;
mapa[6][15] = PAREDE;
mapa[7][6] = PAREDE;
mapa[8][15] = PAREDE;
mapa[9][15] = PAREDE;

mapa[11][9] = PAREDE;
mapa[11][17] = PAREDE;
mapa[11][22] = PAREDE;
mapa[12][1] = PAREDE;
mapa[12][3] = PAREDE;
mapa[12][4] = PAREDE;
mapa[12][9] = PAREDE;
mapa[12][11] = PAREDE;
mapa[12][13] = PAREDE;
mapa[12][17] = PAREDE;
mapa[12][19] = PAREDE;
mapa[12][20] = PAREDE;
mapa[12][22] = PAREDE;
mapa[13][4] = PAREDE;
mapa[13][6] = PAREDE;
mapa[13][7] = PAREDE;
mapa[13][8] = PAREDE;
mapa[13][9] = PAREDE;
mapa[13][11] = PAREDE;
mapa[13][13] = PAREDE;
mapa[13][17] = PAREDE;
mapa[13][20] = PAREDE;
mapa[14][4] = PAREDE;
mapa[14][6] = PAREDE;
mapa[14][11] = PAREDE;
mapa[14][13] = PAREDE;
mapa[14][20] = PAREDE;
mapa[14][21] = PAREDE;
mapa[14][22] = PAREDE;
mapa[15][4] = PAREDE;
mapa[15][5] = PAREDE;
mapa[14][6] = PAREDE;
mapa[15][6] = PAREDE;
mapa[15][11] = PAREDE;
mapa[15][13] = PAREDE;
mapa[15][17] = PAREDE;
mapa[15][18] = PAREDE;
mapa[15][20] = PAREDE;
mapa[15][22] = PAREDE;
mapa[16][11] = PAREDE;
mapa[16][13] = PAREDE;
mapa[16][17] = PAREDE;
mapa[16][22] = PAREDE;
mapa[19][5] = PAREDE;
mapa[20][8] = PAREDE;
mapa[20][10] = PAREDE;
mapa[21][10] = PAREDE;
mapa[21][12] = PAREDE;
mapa[21][22] = PAREDE;
mapa[22][5] = PAREDE;
mapa[22][10] = PAREDE;
mapa[22][14] = PAREDE;
mapa[22][18] = PAREDE;

//Portas
mapa[4][2] = PORTA;
mapa[10][2] = PORTA;
mapa[17][12] = PORTA;

// Chaves
mapa[2][23] = CHAVE;
mapa[9][23] = CHAVE;
mapa[16][23] = CHAVE;

// Caixas
mapa[5][1] = CAIXA;
mapa[5][2] = CAIXA;
mapa[5][3] = CAIXA;
mapa[7][15] = CAIXA;

// Espinhos
mapa[1][2] = ESPINHO;
mapa[1][3] = ESPINHO;
mapa[2][2] = ESPINHO;
mapa[2][3] = ESPINHO;
mapa[1][7] = ESPINHO;
mapa[1][8] = ESPINHO;
mapa[1][13] = ESPINHO;
mapa[1][20] = ESPINHO;
mapa[1][21] = ESPINHO;
mapa[2][5] = ESPINHO;
mapa[2][8] = ESPINHO;
mapa[2][10] = ESPINHO;
mapa[2][15] = ESPINHO;
mapa[2][16] = ESPINHO;
mapa[2][20] = ESPINHO;
mapa[3][6] = ESPINHO;
mapa[3][10] = ESPINHO;
mapa[3][11] = ESPINHO;
mapa[3][13] = ESPINHO;
mapa[3][15] = ESPINHO;
mapa[3][18] = ESPINHO;

mapa[5][4] = ESPINHO;
mapa[5][8] = ESPINHO;
mapa[5][11] = ESPINHO;
mapa[5][12] = ESPINHO;
mapa[6][4] = ESPINHO;
mapa[6][11] = ESPINHO;
mapa[7][1] = ESPINHO;
mapa[7][2] = ESPINHO;
mapa[7][4] = ESPINHO;
mapa[8][6] = ESPINHO;
mapa[8][9] = ESPINHO;
mapa[8][10] = ESPINHO;
mapa[9][3] = ESPINHO;
mapa[9][6] = ESPINHO;
mapa[9][13] = ESPINHO;

mapa[11][7] = ESPINHO;
mapa[12][15] = ESPINHO;
mapa[13][15] = ESPINHO;
mapa[14][15] = ESPINHO;
mapa[15][8] = ESPINHO;
mapa[15][9] = ESPINHO;
mapa[15][15] = ESPINHO;
mapa[15][21] = ESPINHO;

mapa[19][2] = ESPINHO;
mapa[19][18] = ESPINHO;
mapa[19][19] = ESPINHO;
mapa[20][7] = ESPINHO;
mapa[20][15] = ESPINHO;
mapa[21][11] = ESPINHO;
mapa[22][6] = ESPINHO;

// Monstro X
monstroX.x = 19;
monstroX.y = 7;
monstroX.vivo = 1;

mapa[monstroX.y][monstroX.x] = 'X';

// Monstro Y
monstroY.x = 18;
monstroY.y = 14;
monstroY.vivo = 1;

mapa[monstroY.y][monstroY.x] = 'Y';

//BOSS
boss.x = 2;
boss.y = 22;
boss.vivo = 1;
boss.vida = 3;
boss.cooldownataque = 0;

mapa[boss.y][boss.x] = 'Z';
}

// Desenha o mapa na tela.
void desenharmapa()
{
int i, j;

// Limpa a tela.
system("cls || clear");

// Mostra o número da fase.
printf("===== FASE %d =====\n\n", faseatual);

// Percorre o mapa todo.
for(i = 0; i < alturamapa; i++)
{
	for(j = 0; j < larguramapa; j++)
	{
		// Desenha o jogador.
		if(i == player.y && j == player.x)
		{
			printf("%c ", player.dir);
		}
		else
		{
			// Desenha o mapa após uma ação.
			printf("%c ", mapa[i][j]);
		}
	}

	printf("\n");
}

// HUD do player.
printf("\nVidas : %d", player.vidas);
printf("\nChaves: %d\n", player.chaves);

//Arma selecionada pelo jogador
printf("Arma: ");

switch(armaescolhida)
{
case 1:
	printf("Espada\n");
	break;

case 2:
	printf("Arco\n");
	break;

case 3:
	printf("Cajado\n");
	break;

default:
	printf("Nenhuma\n");
}

// Mostra os controles.
printf("\nCONTROLES\n");
printf("WASD = Mover\n");
printf("O = Atacar\n");
printf("I = Interagir\n");
printf("Q = Sair\n");
}

// Movimentação do jogador.
void mover(char tecla)
{
// Posição temporária atual.
int nx = player.x;
int ny = player.y;

// Definição da direção.
switch(tecla)
{
case 'w':
case 'W':
	ny--;
	player.dir = '^';
	break;

case 's':
case 'S':
	ny++;
	player.dir = 'v';
	break;

case 'a':
case 'A':
	nx--;
	player.dir = '<';
	break;

case 'd':
case 'D':
	nx++;
	player.dir = '>';
	break;
}

// Verifica se o player saiu do mapa.
if(nx < 0 || nx >= larguramapa ||
		ny < 0 || ny >= alturamapa)
{
	return;
}

// Bloqueio da parede.
if(mapa[ny][nx] == PAREDE)
	return;

// Bloqueio da porta fechada.
if(mapa[ny][nx] == PORTA)
	return;

// Bloqueio da caixa.
if(mapa[ny][nx] == CAIXA)
	return;

// Espinho retira uma vida.
if(mapa[ny][nx] == ESPINHO)
{
	perdervida();
	return;
}

// Escada troca de fase.
if(mapa[ny][nx] == ESCADA)
{
	if(faseatual == 0 && !armaselecionada)
	{
		printf("\nEscolha uma arma com o NPC primeiro!\n");
		system("pause");
		return;
	}

	// Avança para próxima fase.
	faseatual++;

	// Caso passe da última fase.
	if(faseatual > 3)
	{
		system("cls || clear");

		printf("=================================\n");
		printf("           EPILOGO\n");
		printf("=================================\n\n");

		printf("Com a derrota do Cyclope, o poder\n");
		printf("que controlava a Dungeon foi\n");
		printf("finalmente destruido.\n\n");

		printf("As criaturas restantes fugiram\n");
		printf("para as profundezas e a paz\n");
		printf("retornou ao reino.\n\n");

		printf("Seu nome sera lembrado como o\n");
		printf("aventureiro que conquistou a\n");
		printf("Dungeon.\n\n");

		printf("Creditos:\n");
		printf("Arthur Damaso De Andrade Picanco\n");
		printf("Thiago Sawada Kitajima\n");
		printf("Arthur Moraes De Souza\n");

		system("pause");
		exit(0);
	}

	// Reseta a posição do player.
	player.x = 1;
	player.y = 1;

	// Carrega o mapa novo.
	carregarmapa();

	return;
}

// Atualiza a posição do player.
player.x = nx;
player.y = ny;
}

// Sistema de ataque.
void atacar()
{
if(armaescolhida == 1)
{
	atacarespada();
}
else if(armaescolhida == 2)
{
	atacararco();
}
else if(armaescolhida == 3)
{
	atacarcajado();
}
else
{
	printf("\nEscolha uma arma primeiro!\n");
	system("pause");
}
}

// Morte dos monstros.
void matarmonstro(int x, int y)
{
if(mapa[y][x] == 'X')
{
	mapa[y][x] = CHAO;
	monstroX.vivo = 0;
}

else if(mapa[y][x] == 'Y')
{
	mapa[y][x] = CHAO;
	monstroY.vivo = 0;
}

else if(mapa[y][x] == 'Z')
{

	boss.vida--;

	printf("\nAcertou o Boss! Vida dele: %d\n", boss.vida);
	system("pause");

	if(boss.vida <= 0)
	{
		mapa[y][x] = CHAO;
		boss.vivo = 0;

		printf("\nO Cyclope foi derrotado!\n");
		system("pause");
		mapa[23][23] = ESCADA;
	}
}
}

//Ataque da espada.
void atacarespada()
{
int x, y;

if(player.dir == '^')
{
	for(y = player.y - 2; y <= player.y - 1; y++)
	{
		for(x = player.x - 1; x <= player.x + 1; x++)
		{
			if(x >= 0 && x < larguramapa &&
					y >= 0 && y < alturamapa)
			{
				if(mapa[y][x] == PAREDE)
					continue;

				if(mapa[y][x] == CAIXA)
				{
					mapa[y][x] = CHAO;
				}

				matarmonstro(x, y);
			}
		}
	}
}

else if(player.dir == 'v')
{
	for(y = player.y + 1; y <= player.y + 2; y++)
	{
		for(x = player.x - 1; x <= player.x + 1; x++)
		{
			if(x >= 0 && x < larguramapa &&
					y >= 0 && y < alturamapa)
			{
				if(mapa[y][x] == PAREDE)
					continue;

				if(mapa[y][x] == CAIXA)
				{
					mapa[y][x] = CHAO;
				}

				matarmonstro(x, y);
			}
		}
	}
}

else if(player.dir == '<')
{
	for(x = player.x - 2; x <= player.x - 1; x++)
	{
		for(y = player.y - 1; y <= player.y + 1; y++)
		{
			if(x >= 0 && x < larguramapa &&
					y >= 0 && y < alturamapa)
			{
				if(mapa[y][x] == PAREDE)
					continue;

				if(mapa[y][x] == CAIXA)
				{
					mapa[y][x] = CHAO;
				}

				matarmonstro(x, y);
			}
		}
	}
}

else if(player.dir == '>')
{
	for(x = player.x + 1; x <= player.x + 2; x++)
	{
		for(y = player.y - 1; y <= player.y + 1; y++)
		{
			if(x >= 0 && x < larguramapa &&
					y >= 0 && y < alturamapa)
			{
				if(mapa[y][x] == PAREDE)
					continue;

				if(mapa[y][x] == CAIXA)
				{
					mapa[y][x] = CHAO;
				}

				matarmonstro(x, y);
			}
		}
	}
}

printf("\nGolpe de espada executado!\n");
system("pause");
}

//Ataque do arco.
void atacararco()
{
int i;
int x;
int y;

for(i = 1; i <= 4; i++)
{
	x = player.x;
	y = player.y;

	if(player.dir == '^')
		y -= i;
	else if(player.dir == 'v')
		y += i;
	else if(player.dir == '<')
		x -= i;
	else if(player.dir == '>')
		x += i;

	if(x < 0 || x >= larguramapa || y < 0 || y >= alturamapa)
		break;

	if(mapa[y][x] == PAREDE)
		break;

	if(mapa[y][x] == CAIXA)
		mapa[y][x] = CHAO;

	matarmonstro(x, y);
}

printf("\nFlecha disparada!\n");
system("pause");
}

//Ataque do cajado.
void atacarcajado()
{
int x;
int y;

for(y = player.y - 1; y <= player.y + 1; y++)
{
	for(x = player.x - 1; x <= player.x + 1; x++)
	{
		if(x < 0 || x >= larguramapa ||
				y < 0 || y >= alturamapa)
		{
			continue;
		}

		if(mapa[y][x] == PAREDE)
		{
			continue;
		}

		if(x == player.x && y == player.y)
		{
			continue;
		}

		if(mapa[y][x] == CAIXA)
		{
			mapa[y][x] = CHAO;
		}

		matarmonstro(x, y);
	}
}

printf("\nMagia do cajado utilizada!\n");
system("pause");
}

// Sistema de interação.
void interagir()
{
// Posição da interação.
int tx = player.x;
int ty = player.y;

// Direção da interação.
switch(player.dir)
{
case '^':
	ty--;
	break;

case 'v':
	ty++;
	break;

case '<':
	tx--;
	break;

case '>':
	tx++;
	break;
}

// Verifica os limites do mapa para interação.
if(tx < 0 || tx >= larguramapa ||
		ty < 0 || ty >= alturamapa)
{
	return;
}

// Pegar chave.
if(mapa[ty][tx] == CHAVE)
{
	mapa[ty][tx] = CHAO;

	player.chaves++;

	printf("\nVoce pegou uma chave!\n");

	system("pause");
}

// Abrir a porta.
else if(mapa[ty][tx] == PORTA)
{
	// Verifica se o player tem chaves.
	if(player.chaves > 0)
	{
		mapa[ty][tx] = PORTA_ABERTA;

		player.chaves--;

		printf("\nPorta aberta!\n");
	}
	else
	{
		printf("\nVoce precisa de uma chave!\n");
	}

	system("pause");
}

// Interação com o NPC.
else if(mapa[ty][tx] == NPC)
{
	int opcao;

	printf("\n===== ESCOLHA SUA ARMA =====\n\n");

	printf("1 - Espada\n");
	printf("2 - Arco\n");
	printf("3 - Cajado\n");

	printf("\nEscolha: ");
	scanf("%d", &opcao);

	switch(opcao)
	{
	case 1:
		armaescolhida = 1;
		printf("\nVoce escolheu ESPADA!\n");
		break;

	case 2:
		armaescolhida = 2;
		printf("\nVoce escolheu ARCO!\n");
		break;

	case 3:
		armaescolhida = 3;
		printf("\nVoce escolheu CAJADO!\n");
		break;

	default:
		printf("\nOpcao invalida!\n");
		system("pause");
		return;
	}

	armaselecionada = 1;

	printf("\nAgora voce pode entrar na dungeon!\n");

	system("pause");
}

else if(mapa[ty][tx] == BOTAO)
{
	if(botaoativado)
	{
		printf("\nO botao ja foi ativado!\n");
		system("pause");
		return;
	}

	botaoativado = 1;

	printf("\nBotao pressionado!\n");

	mapa[2][1] = CHAO;
	mapa[2][2] = CHAO;
	mapa[2][3] = CHAO;
	mapa[2][4] = CHAO;
	mapa[2][5] = CHAO;
	mapa[2][6] = CHAO;
	mapa[2][7] = CHAO;
	mapa[2][8] = CHAO;
	mapa[2][9] = CHAO;
	mapa[2][10] = CHAO;
	mapa[2][11] = CHAO;
	mapa[2][12] = CHAO;
	mapa[2][13] = CHAO;

	printf("A parede abaixou!\n");

	system("pause");

}
}

// Sistema de perder vida.
void perdervida()
{
// Remove uma vida.
player.vidas--;

// Game Over.
if(player.vidas <= 0)
{
	system("cls");

	printf("\n===== GAME OVER =====\n\n");

	system("pause");

	gameover = 1;
	return;
}

// Mensagem de dano.
printf("\nVoce perdeu uma vida!\n");

// Reposicionar o player após um dano e reinicia a fase.
player.x = 1;
player.y = 1;

carregarmapa();

system("pause");
}

// IA do monstro X
void movermonstroX()
{
int dx[4] = {0, 0, -1, 1};
int dy[4] = { -1, 1, 0, 0};

int dir = rand() % 4;

int nx = monstroX.x + dx[dir];
int ny = monstroX.y + dy[dir];

if(!monstroX.vivo)
	return;

if(mapa[ny][nx] != CHAO)
	return;

mapa[monstroX.y][monstroX.x] = CHAO;

monstroX.x = nx;
monstroX.y = ny;

mapa[monstroX.y][monstroX.x] = 'X';
}

//IA do monstro Y
void movermonstroY()
{
int nx = monstroY.x;
int ny = monstroY.y;

if(!monstroY.vivo)
	return;

if(abs(player.x - monstroY.x) >
		abs(player.y - monstroY.y))
{
	if(player.x > monstroY.x)
		nx++;
	else if(player.x < monstroY.x)
		nx--;
}
else
{
	if(player.y > monstroY.y)
		ny++;
	else if(player.y < monstroY.y)
		ny--;
}

if(mapa[ny][nx] != CHAO)
	return;

mapa[monstroY.y][monstroY.x] = CHAO;

monstroY.x = nx;
monstroY.y = ny;

mapa[monstroY.y][monstroY.x] = 'Y';
}

//IA do boss
void moverboss()
{
if(!boss.vivo)
	return;

int nx = boss.x;
int ny = boss.y;

//Ataque em três turnos
boss.cooldownataque++;

if(boss.cooldownataque >= 3)
{
	boss.cooldownataque = 0;

	printf("\nBOSS USOU ATAQUE EM AREA!\n");

	int x, y;

	for(y = boss.y - 1; y <= boss.y + 1; y++)
	{
		for(x = boss.x - 1; x <= boss.x + 1; x++)
		{
			if(x < 0 || x >= larguramapa ||
					y < 0 || y >= alturamapa)
				continue;

			if(x == player.x && y == player.y)
			{
				printf("\nVOCE FOI ATINGIDO!\n");
				perdervida();
			}
		}
	}

	system("pause");
	return;
}

// Perseguição do boss.
int velocidade = 1;

// fica mais rápido quando vida <= 1.
if(boss.vida <= 1)
	velocidade = 2;

int i;

for(i = 0; i < velocidade; i++)
{
	if(player.x > boss.x) nx++;
	else if(player.x < boss.x) nx--;

	if(player.y > boss.y) ny++;
	else if(player.y < boss.y) ny--;
}

// Colisão com mapa.
if(nx < 0 || nx >= larguramapa ||
		ny < 0 || ny >= alturamapa)
	return;

if(mapa[ny][nx] != CHAO)
	return;

mapa[boss.y][boss.x] = CHAO;

boss.x = nx;
boss.y = ny;

mapa[boss.y][boss.x] = 'Z';

// Contato direto.
if(boss.x == player.x && boss.y == player.y)
{
	printf("\nBOSS TE ACERTOU!\n");
	perdervida();
}
}

// Variável universal para mover monstros.
void movermonstros()
{
movermonstroX();
movermonstroY();
moverboss();

if(monstroX.vivo &&
		monstroX.x == player.x &&
		monstroX.y == player.y)
{
	perdervida();
}

if(monstroY.vivo &&
		monstroY.x == player.x &&
		monstroY.y == player.y)
{
	perdervida();
}
}

// Tela de tutorial.
void tutorial()
{
system("cls");

printf("===== TUTORIAL =====\n\n");

// Explicação dos símbolos.
printf("SIMBOLOS:\n\n");

printf("* = Parede\n");
printf(". = Chao\n");
printf("# = Espinho\n");
printf("k = Caixa\n");
printf("@ = Chave\n");
printf("D = Porta Fechada\n");
printf("= = Porta Aberta\n");
printf("L = Escada\n");
printf("O = Botão\n");
printf("N = NPC\n");
printf("Z = Boss(Cyclope)\n\n");
printf("X = Monstro X(Goblin)\n");
printf("Y = Monstro Y(Esqueleto)\n");

// Explicação dos controles.
printf("CONTROLES:\n\n");

printf("w = Cima\n");
printf("a = Esquerda\n");
printf("s = Baixo\n");
printf("d = Direita\n");
printf("o = Atacar\n");
printf("I = Interagir\n");
printf("Q = Sair\n");

// Espera o enter para retornar ao menu.
printf("\nPressione ENTER para voltar ao menu principal...");
getchar();
getchar();
}
