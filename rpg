/*
 * MIRABILIA - RPG de tabuleiro em modo texto (C99)
 * Cada fase tem 12 casas: o d6 move o heroi e a casa dispara um evento.
 * Os status sao alterados por funcoes que recebem ponteiros.
 */

#include <stdio.h>
#include <stdlib.h>
#include <time.h>

#define NUM_FASES          3
#define CASAS_POR_FASE     12
#define MONSTROS_POR_FASE  3
#define NIVEL_MAXIMO       7
/* desvio: d20 + esquiva >= LIMITE_DESVIO */
#define LIMITE_DESVIO      15
#define LIMITE_REVIVER     10
#define LIMITE_FUGA        10
#define LIMITE_FUGA_FERIDO 14
#define XP_TESOURO         3
#define CURA_POCAO         10
#define LIMITE_POCAO_BAU   15

typedef enum { CLASSE_EXPLORADOR = 1, CLASSE_PREDADOR, CLASSE_ESPECIALISTA } ClassePersonagem;

typedef enum { CASA_VAZIA, CASA_COMBATE, CASA_TESOURO, CASA_CURA, CASA_SAIDA } TipoCasa;

typedef enum { ACAO_ATACAR = 1, ACAO_HABILIDADE, ACAO_DEFENDER, ACAO_FUGIR } AcaoCombate;

typedef enum { MENU_ROLAR = 1, MENU_STATUS, MENU_BOLSA } AcaoTabuleiro;

typedef enum { COMBATE_VITORIA, COMBATE_FUGA, COMBATE_DERROTA } ResultadoCombate;

typedef enum { DESVIO_CRITICO, DESVIO_OK, DESVIO_FALHA, DESVIO_FALHA_CRITICA } ResultadoDesvio;

/* dano = base + qtd dados de 'lados' faces. Ex: 4 + 2d6 -> {4, 2, 6} */
typedef struct {
    int base;
    int qtd;
    int lados;
} Dano;

typedef struct {
    char nome[30];
    ClassePersonagem classe;
    char nomeClasse[20];
    char arma[50];
    char armadura[50];
    int vida, vidaMax;
    int forca, agilidade, inteligencia, defesa;
    int nivel, xp;
    int pocoes;
    Dano ataqueBasico;
    char nomeHabilidade[30];
    Dano habilidade;
    int recargaHabilidade;
    int recargaAtual;
    int sugaCargas;
} Personagem;

typedef struct {
    char nome[30];
    char descricao[130];
    int vida, vidaMax;
    int esquiva;
    int xpRecompensa;
    Dano ataque;
    char textoAtaque[120];
    int temEspecial;
    char nomeEspecial[30];
    Dano especial;
    int recargaEspecial;
    int recargaAtual;
    char textoEspecial[120];
} Monstro;

typedef struct {
    char nome[30];
    char missao[100];
    Monstro monstros[MONSTROS_POR_FASE];
} Fase;

void limparBuffer() {
    int c;
    c = getchar();
    while (c != '\n' && c != EOF) {
        c = getchar();
    }
}

void pausar() {
    printf("\n[Pressione ENTER para continuar...]\n");
    limparBuffer();
}

int lerOpcao(char mensagem[], int min, int max) {
    int valor, lidos;

    while (1) {
        printf("%s", mensagem);
        lidos = scanf("%d", &valor);

        limparBuffer();

        if (lidos == 1 && valor >= min && valor <= max) {
            return valor;
        }
        printf("Opcao invalida! Digite um numero entre %d e %d.\n", min, max);
    }
}

int rolarDado(int lados) {
    return rand() % lados + 1;
}

int rolarDano(Dano d) {
    int total, i;
    total = d.base;
    for (i = 0; i < d.qtd; i++) {
        total = total + rolarDado(d.lados);
    }
    return total;
}

int rolarDadoJogador(Personagem *p, int lados) {
    int resultado;
    resultado = rolarDado(lados);
    if (p->classe == CLASSE_ESPECIALISTA && resultado == 1) {
        printf("  [Passiva] Saiu 1! Voce rola o dado de novo.\n");
        resultado = rolarDado(lados);
    }
    return resultado;
}

int xpParaSubir(int nivel) {
    switch (nivel) {
        case 1: return 3;
        case 2: return 5;
        case 3: return 7;
        case 4: return 9;
        case 5: return 11;
        case 6: return 13;
        default: return 0;
    }
}

/* Desperto (nivel 2+): +1 de dano a cada 5 de inteligencia */
int bonusDesperto(Personagem *p) {
    if (p->nivel >= 2) {
        return p->inteligencia / 5;
    }
    return 0;
}

/* esquiva: agilidade/10 no nivel 1 e agilidade/5 a partir do nivel 2 */
int calcularEsquiva(Personagem *p) {
    if (p->nivel >= 2) {
        return p->agilidade / 5;
    }
    return p->agilidade / 10;
}

int calcularDanoJogador(Personagem *p, int especial) {
    Dano d;
    int total, i;

    if (especial == 1) {
        d = p->habilidade;
    } else {
        d = p->ataqueBasico;
    }

    total = d.base + bonusDesperto(p);
    if (especial == 0) {
        total = total + p->forca / 5;
    }
    for (i = 0; i < d.qtd; i++) {
        total = total + rolarDadoJogador(p, d.lados);
    }
    return total;
}

void receberDano(Personagem *p, int dano) {
    p->vida = p->vida - dano;
    if (p->vida < 0) {
        p->vida = 0;
    }
}

void danoNoMonstro(Monstro *m, int dano) {
    m->vida = m->vida - dano;
    if (m->vida < 0) {
        m->vida = 0;
    }
}

int curar(Personagem *p, int quantia) {
    int vidaAntes;
    vidaAntes = p->vida;
    p->vida = p->vida + quantia;
    if (p->vida > p->vidaMax) {
        p->vida = p->vidaMax;
    }
    return p->vida - vidaAntes;
}

void subirDeNivel(Personagem *p) {
    int bonus, sorteio;

    p->nivel = p->nivel + 1;
    p->xp = 0;
    printf("\n*** VOCE SUBIU PARA O NIVEL %d! ***\n", p->nivel);

    /* principal +5; +2 sorteado em um dos outros dois atributos */
    sorteio = rolarDado(2);

    if (p->classe == CLASSE_EXPLORADOR) {
        p->agilidade = p->agilidade + 5;
        if (sorteio == 1) {
            p->forca = p->forca + 2;
            printf("Bonus aleatorio: +2 de Forca!\n");
        } else {
            p->inteligencia = p->inteligencia + 2;
            printf("Bonus aleatorio: +2 de Inteligencia!\n");
        }
    } else if (p->classe == CLASSE_PREDADOR) {
        p->forca = p->forca + 5;
        if (sorteio == 1) {
            p->agilidade = p->agilidade + 2;
            printf("Bonus aleatorio: +2 de Agilidade!\n");
        } else {
            p->inteligencia = p->inteligencia + 2;
            printf("Bonus aleatorio: +2 de Inteligencia!\n");
        }
    } else {
        p->inteligencia = p->inteligencia + 5;
        if (sorteio == 1) {
            p->forca = p->forca + 2;
            printf("Bonus aleatorio: +2 de Forca!\n");
        } else {
            p->agilidade = p->agilidade + 2;
            printf("Bonus aleatorio: +2 de Agilidade!\n");
        }
    }
    printf("Forca: %d | Agilidade: %d | Inteligencia: %d\n",
           p->forca, p->agilidade, p->inteligencia);

    if (p->nivel == 2) {
        printf("Voce despertou! (Desperto: mais vida, esquiva e dano pela inteligencia)\n");
    }
    if (p->nivel >= 2) {
        bonus = p->vidaMax / 5;
        p->vidaMax = p->vidaMax + bonus;
        p->vida = p->vida + bonus;
        printf("Bonus Desperto: +%d de vida maxima (agora %d).\n", bonus, p->vidaMax);
    }
}

void ganharXP(Personagem *p, int xp) {
    if (p->nivel >= NIVEL_MAXIMO) {
        printf("Voce ja esta no nivel maximo!\n");
        return;
    }
    p->xp = p->xp + xp;
    printf("+%d XP  (%d/%d para o proximo nivel)\n", xp, p->xp, xpParaSubir(p->nivel));
    if (p->xp >= xpParaSubir(p->nivel)) {
        subirDeNivel(p);
    }
}

int tentarReviver(Personagem *p) {
    int tentativas, i, d20;

    tentativas = 3;
    if (p->classe == CLASSE_PREDADOR) {
        tentativas = 4;   /* passiva do Predador */
    }

    printf("\n*** Sua vida chegou a 0! Voce desmaiou... ***\n");
    printf("Voce tem %d tentativa(s) de reviver (d20 >= %d).\n", tentativas, LIMITE_REVIVER);

    for (i = 1; i <= tentativas; i++) {
        d20 = rolarDado(20);
        printf("  Tentativa %d: d20 = %d\n", i, d20);
        if (d20 >= LIMITE_REVIVER) {
            p->vida = p->vidaMax / 4;
            if (p->vida < 1) {
                p->vida = 1;
            }
            printf("Voce abre os olhos e se levanta com %d de vida!\n", p->vida);
            return 1;
        }
    }
    return 0;
}

/* valores na ordem dos campos da struct Personagem */
void criarPersonagem(Personagem *p, int classe) {
    Personagem explorador = {
        "", CLASSE_EXPLORADOR, "Explorador", "Faca de combate do DCA", "Colete tatico leve",
        20, 20,
        15, 25, 10, 2,
        1, 0, 0,
        {4, 1, 6},
        "Apunhalar", {4, 2, 6}, 3,
        0, 0
    };
    Personagem predador = {
        "", CLASSE_PREDADOR, "Predador", "Manoplas de contencao", "Armadura reforcada do DCA",
        35, 35,
        25, 15, 10, 3,
        1, 0, 0,
        {3, 1, 6},
        "Suga Vida", {0, 0, 0}, 6,
        0, 0
    };
    Personagem especialista = {
        "", CLASSE_ESPECIALISTA, "Especialista", "Emissor de energia anomala",
        "Jaleco-uniforme com nucleo anomalo",
        15, 15,
        10, 15, 25, 2,
        1, 0, 0,
        {5, 1, 6},
        "Energia de Anomalia", {5, 1, 8}, 2,
        0, 0
    };

    if (classe == CLASSE_EXPLORADOR) {
        *p = explorador;
    } else if (classe == CLASSE_PREDADOR) {
        *p = predador;
    } else {
        *p = especialista;
    }
}

/* valores na ordem dos campos da struct Monstro */
void inicializarFases(Fase fases[]) {
    Monstro leaoMarinho = {
        "Leao Marinho",
        "Um leao-marinho de circo, brilhante e corrompido, que se arrasta pelo asfalto rachado.",
        12, 12, 3, 2, {0, 1, 4},
        "O Leao Marinho se arremessa contra voce com a nadadeira!",
        0, "", {0, 0, 0}, 0, 0, ""
    };
    Monstro cachorro = {
        "Cachorro",
        "Um cachorro adestrado de olhos magenta, ainda obedecendo comandos que ninguem da.",
        15, 15, 5, 3, {1, 1, 4},
        "O Cachorro salta e crava os dentes em voce!",
        0, "", {0, 0, 0}, 0, 0, ""
    };
    Monstro leao = {
        "Leao",
        "Um leao imenso de juba desfiada, que ruge como se ainda houvesse plateia.",
        18, 18, 6, 4, {2, 1, 6},
        "O Leao ruge e golpeia voce com as garras!",
        0, "", {0, 0, 0}, 0, 0, ""
    };

    Monstro trapezista = {
        "Trapezista",
        "Uma acrobata de membros longos que se pendura no teto do tunel.",
        14, 14, 10, 5, {0, 1, 4},
        "A Trapezista despenca do teto e chuta voce!",
        0, "", {0, 0, 0}, 0, 0, ""
    };
    Monstro monociclista = {
        "Monociclista",
        "Um monociclista que pedala em alta velocidade sem nunca cair.",
        14, 14, 11, 6, {0, 1, 6},
        "O Monociclista passa a toda velocidade e atropela voce!",
        0, "", {0, 0, 0}, 0, 0, ""
    };
    Monstro dancarina = {
        "Dancarina",
        "Uma bailarina de sorriso congelado cujos passos fazem o chao tremer.",
        14, 14, 12, 7, {0, 1, 8},
        "A Dancarina gira e acerta voce com uma pirueta cortante!",
        0, "", {0, 0, 0}, 0, 0, ""
    };

    Monstro contorcionista = {
        "Contorcionista",
        "Uma artista cujo corpo se dobra em angulos impossiveis.",
        14, 14, 13, 8, {0, 1, 6},
        "A Contorcionista se desdobra e golpeia voce de um angulo impossivel!",
        0, "", {0, 0, 0}, 0, 0, ""
    };
    Monstro magico = {
        "Magico",
        "Um mago de cartola e capa rasgada, cheio de truques mortais.",
        15, 15, 10, 9, {1, 1, 4},
        "O Magico saca uma carta afiada e arremessa em voce!",
        1, "Bombas da Cartola", {1, 2, 4}, 4, 0,
        "O Magico tira varias bombas da cartola e joga todas em voce!"
    };
    Monstro palhaco = {
        "Palhaco",
        "O anfitriao do circo. Seu sorriso nunca some, mas seus olhos carregam um peso antigo.",
        20, 20, 12, 10, {2, 1, 6},
        "O Palhaco acerta voce com um martelo de borracha reforcado!",
        1, "Torta Toxica", {1, 2, 6}, 3, 0,
        "O Palhaco arremessa uma torta toxica bem no seu rosto!"
    };

    Fase fase1 = {
        "Zona de Quarentena",
        "Atravessar a Zona de Quarentena e encontrar o bueiro que leva ao subsolo.",
        {leaoMarinho, cachorro, leao}
    };
    Fase fase2 = {
        "Tuneis do Riso",
        "Atravessar os Tuneis do Riso seguindo o eco das risadas ate a lona do circo.",
        {trapezista, monociclista, dancarina}
    };
    Fase fase3 = {
        "O Circo",
        "Chegar ao centro da arena e enfrentar o Palhaco.",
        {contorcionista, magico, palhaco}
    };

    fases[0] = fase1;
    fases[1] = fase2;
    fases[2] = fase3;
}

void mostrarIntroducaoFase(int numeroFase) {
    if (numeroFase == 1) {
        printf("As ruas da zona de quarentena estao vazias e cobertas de poeira rosada.\n");
        printf("Cartazes antigos e jaulas quebradas denunciam que algo estranho passou por aqui.\n");
        printf("Seu radio chia: 'Agente, siga em frente. O bueiro no fim da rua e a unica saida.'\n");
    } else if (numeroFase == 2) {
        printf("Os tuneis do subsolo sao estreitos e umidos. Gargalhadas ecoam nas paredes,\n");
        printf("mas nao ha alegria nelas. Quanto mais voce avanca, mais alto elas ficam.\n");
    } else {
        printf("A lona do circo brilha em magenta. As arquibancadas estao vazias, mas voce sente\n");
        printf("centenas de olhos te observando. No centro da arena, alguem espera por voce.\n");
    }
}

void mostrarSaidaFase(int numeroFase) {
    if (numeroFase == 1) {
        printf("Voce encontra o bueiro. La de baixo vem um som baixinho... parece uma risada.\n");
        printf("Voce respira fundo e desce para os tuneis.\n");
    } else if (numeroFase == 2) {
        printf("No fim do tunel, uma cortina vermelha e rosa se abre sozinha.\n");
        printf("Do outro lado, uma enorme lona de circo se ergue sob um teto de pedra.\n");
    }
}

void mostrarHistoriaCasaVazia(int numeroFase, int indice) {
    if (numeroFase == 1) {
        if (indice == 0) {
            printf("Um cartaz rasgado: 'CIRCO MAGENTA - o riso que cura'. A tinta ainda parece\n");
            printf("fresca, mesmo depois de duzentos anos.\n");
        } else if (indice == 1) {
            printf("Jaulas enferrujadas, todas abertas POR DENTRO. Nao ha sinais de arrombamento...\n");
            printf("quem estava preso ali saiu por conta propria?\n");
        } else {
            printf("Um bilhete amassado: 'Se o DCA vier, nao reajam. Somos so artistas.'\n");
            printf("O resto esta manchado e ilegivel.\n");
        }
    } else if (numeroFase == 2) {
        if (indice == 0) {
            printf("As paredes do tunel ecoam gargalhadas distantes. Nao parecem de alegria.\n");
        } else if (indice == 1) {
            printf("Rabiscos de giz: criancas, estrelas, uma lona vermelha e rosa. Alguem desenhava\n");
            printf("aqui para nao esquecer o caminho de casa.\n");
        } else {
            printf("Um sapato gigante de palhaco e, atras dele, um cartaz: 'Espers sao bem-vindos'.\n");
            printf("Alguem raspou o resto do texto.\n");
        }
    } else {
        if (indice == 0) {
            printf("Arquibancadas vazias. Cada assento tem um nome gravado: eram familias, nao plateia.\n");
        } else if (indice == 1) {
            printf("No camarim, espelhos rachados refletem rostos sem maquiagem. Todos sorriem para voce.\n");
        } else {
            printf("Uma placa torta: 'Aqui acolhemos quem despertou. Aqui ninguem luta sozinho.'\n");
            printf("Embaixo dela, marcas de um desabamento.\n");
        }
    }
}

void revelacaoDoPalhaco() {
    printf("\nO Palhaco cai de joelhos. A maquiagem derrete e, por um instante,\n");
    printf("voce enxerga o rosto de um homem cansado.\n");
    pausar();
    printf("Palhaco: 'Voces nos chamam de anomalias... mas ja fomos gente.'\n");
    printf("Palhaco: 'Todos aqui eram Espers. Fizemos deste circo um grupo de apoio e\n");
    printf("         ajuda: um lugar onde ninguem despertava sozinho.'\n");
    pausar();
    printf("Palhaco: 'O DCA nao gostou. Disseram que juntos eramos perigosos.\n");
    printf("         Soterraram o circo inteiro conosco dentro.'\n");
    printf("Palhaco: 'O que sobrou somos nos: anomalias com as memorias de quem fomos.'\n");
    pausar();
    printf("Ele sorri pela ultima vez e se desfaz em poeira rosada.\n");
    printf("Voce ergue os olhos. A lona do circo esta em silencio.\n");
    printf("Agora voce sabe a verdade sobre o DCA.\n");
}

void mostrarStatus(Personagem *p) {
    printf("\n========== STATUS ==========\n");
    printf("%s - %s\n", p->nome, p->nomeClasse);
    printf("Nivel: %d", p->nivel);
    if (p->nivel >= 2) {
        printf(" (Desperto)");
    }
    if (p->nivel < NIVEL_MAXIMO) {
        printf("   XP: %d/%d\n", p->xp, xpParaSubir(p->nivel));
    } else {
        printf("   (nivel maximo)\n");
    }
    printf("Vida: %d/%d\n", p->vida, p->vidaMax);
    printf("Forca: %d | Agilidade: %d | Inteligencia: %d | Esquiva: %d | Defesa: %d\n",
           p->forca, p->agilidade, p->inteligencia, calcularEsquiva(p), p->defesa);
    printf("Arma: %s\nArmadura: %s\n", p->arma, p->armadura);
    printf("Habilidade especial: %s\n", p->nomeHabilidade);
    printf("============================\n");
}

void abrirBolsa(Personagem *p) {
    int curado;

    printf("\n----- BOLSA -----\n");
    printf("Pocoes de cura: %d (cada uma recupera %d de vida)\n", p->pocoes, CURA_POCAO);
    if (p->pocoes == 0) {
        printf("Voce nao tem itens de cura.\n");
        return;
    }
    if (p->vida == p->vidaMax) {
        printf("Sua vida ja esta cheia.\n");
        return;
    }
    if (lerOpcao("Usar uma pocao? [1] Sim  [2] Nao: ", 1, 2) == 1) {
        p->pocoes = p->pocoes - 1;
        curado = curar(p, CURA_POCAO);
        printf("Voce bebe a pocao e recupera %d de vida. (Vida: %d/%d)\n", curado, p->vida, p->vidaMax);
    }
}

void introducao() {
    printf("==============================================================\n");
    printf("                         MIRABILIA\n");
    printf("==============================================================\n");
    printf("200 anos atras, a cidade de Mirabilia foi atingida pelo fenomeno conhecido\n");
    printf("como 'A Noite Magenta'. Nao se sabe de sua origem, se foi algo sobrenatural\n");
    printf("ou vindo de outro mundo, mas neste dia a historia das pessoas daquela grande\n");
    printf("metropole iria mudar.\n");
    printf("O brilho rosa, visivel apenas naquela cidade, encantou os olhos de todos os que\n");
    printf("olharam para cima. Algo nunca visto antes encantava uns e assustava outros.\n");
    pausar();

    printf("Junto com aquele ceu fora do comum, algo estranho comecou a aparecer pela cidade.\n");
    printf("Criaturas estranhas passaram a caminhar pelas ruas. Certamente nao eram seres\n");
    printf("amigaveis, muito menos bonitos.\n");
    printf("Humanos que cruzavam o caminho dessas criaturas encontravam seu fim em instantes.\n");
    printf("Alguns nao morriam, mas enlouqueciam...\n");
    printf("Nem todas essas criaturas eram crueis e sanguinarias: cada uma tinha sua\n");
    printf("particularidade.\n");
    pausar();

    printf("Alem de trazer essas anomalias, o ceu magenta deixou os humanos vulneraveis a sua\n");
    printf("influencia. Alguns enlouqueceram ou, pior, se transformaram em anomalias.\n");
    printf("Pais viam seus filhos perderem a humanidade diante dos olhos, jovens perdiam\n");
    printf("seus melhores amigos para a loucura, e a vida virou um caos...\n");
    pausar();

    printf("Da mesma forma que pessoas viraram anomalias, nasceram aqueles humanos que\n");
    printf("conseguiram conter a energia anomala vinda do ceu.\n");
    printf("Com a queda e a insuficiencia das forcas policiais, esses humanos especiais\n");
    printf("tomaram a linha de frente por vontade propria e ajudaram a livrar a cidade dos\n");
    printf("males da Noite Magenta.\n");
    printf("Assim, aquela noite e os dias seguintes viram uma fagulha de esperanca.\n");
    printf("O ceu voltou ao normal, mas a vida das pessoas da cidade, nao.\n");
    pausar();

    printf("============================ HOJE ============================\n");
    printf("Dois seculos apos a Noite Magenta, Mirabilia vive de forma totalmente diferente.\n");
    printf("Hoje existe o DCA, Departamento de Combate a Anomalias, criado por aqueles que\n");
    printf("aprenderam a controlar a exposicao a forca anomala vinda do alem.\n");
    printf("Leis e regras foram criadas pelo governo da cidade, pela policia, pelo DCA e\n");
    printf("com o exterior.\n");
    pausar();

    printf("Primeiro: a cidade esta isolada do resto do mundo. Apenas pessoas autorizadas\n");
    printf("podem sair, e quem entra nao pode sair, a menos que seja autorizado.\n");
    printf("O DCA e a forca de seguranca mais importante da cidade: cuida dos casos de\n");
    printf("anomalia, que sao frequentes, enquanto a policia cuida dos casos civis simples.\n");
    printf("Pessoas que despertam poderes sao chamadas de 'Espers'. Podem despertar a\n");
    printf("qualquer momento da vida e, por lei, sao obrigadas a servir ao DCA.\n");
    printf("Quem nao despertou tambem pode servir ao DCA e contar com o apoio tecnologico\n");
    printf("do departamento para se tornar um combatente.\n");
    pausar();

    printf("Hoje, apos alguns meses de treinamento, voce esta pronto para sua primeira\n");
    printf("missao na zona de quarentena. Que tipo de combatente voce e?\n");
    pausar();
}

void escolherPersonagem(Personagem *p) {
    int classe;

    printf("==============================================================\n");
    printf("ESCOLHA SUA CLASSE NO DCA:\n");
    printf("1 - Explorador   (Alta agilidade - fuga e reconhecimento)\n");
    printf("                 Vida 20 | Forca 15 | Agilidade 25 | Inteligencia 10 | Defesa 2\n");
    printf("2 - Predador     (Alta forca e vida - linha de frente)\n");
    printf("                 Vida 35 | Forca 25 | Agilidade 15 | Inteligencia 10 | Defesa 3\n");
    printf("3 - Especialista (Alta inteligencia - combate a distancia)\n");
    printf("                 Vida 15 | Forca 10 | Agilidade 15 | Inteligencia 25 | Defesa 2\n");

    classe = lerOpcao("\nDigite o numero da sua classe (1, 2 ou 3): ", 1, 3);
    criarPersonagem(p, classe);

    printf("Qual e o seu nome, agente (uma palavra so)? ");
    scanf("%29s", p->nome);
    limparBuffer();

    printf("\nAgente %s designado. Aqui esta o que voce precisa!\n", p->nomeClasse);
    printf("Arma: %s\nArmadura: %s\n", p->arma, p->armadura);
    if (p->classe == CLASSE_EXPLORADOR) {
        printf("Habilidade: Apunhalar (4 + 2d6, recarga 3 turnos)\n");
        printf("Passiva: se falhar em desviar ou fugir, pode tentar mais uma vez.\n");
    } else if (p->classe == CLASSE_PREDADOR) {
        printf("Habilidade: Suga Vida (seus proximos 3 ataques curam 50%% do dano, recarga 6 turnos)\n");
        printf("Passiva: ganha uma tentativa extra para reviver.\n");
    } else {
        printf("Habilidade: Energia de Anomalia (5 + 1d8, recarga 2 turnos)\n");
        printf("Passiva: nunca erra criticamente e pode rolar de novo um dado que falhou ou deu 1.\n");
    }
    pausar();
}

void mostrarCombate(Personagem *p, Monstro *m) {
    printf("\n------------------------------------------------\n");
    printf("%s: %d/%d de vida   |   %s: %d/%d de vida\n",
           p->nome, p->vida, p->vidaMax, m->nome, m->vida, m->vidaMax);
    printf("------------------------------------------------\n");
}

int escolherAcao(Personagem *p, int podeFugir) {
    int opcao;

    while (1) {
        printf("\nSua acao:\n  [1] Atacar\n");
        if (p->recargaAtual > 0) {
            printf("  [2] %s (recarregando: %d turno(s))\n", p->nomeHabilidade, p->recargaAtual);
        } else {
            printf("  [2] %s\n", p->nomeHabilidade);
        }
        printf("  [3] Defender (reduz o dano do proximo ataque)\n");
        if (podeFugir == 1) {
            printf("  [4] Fugir\n");
        } else {
            printf("  [4] Fugir (impossivel neste combate)\n");
        }

        opcao = lerOpcao("Escolha: ", 1, 4);

        if (opcao == ACAO_HABILIDADE && p->recargaAtual > 0) {
            printf("Essa habilidade ainda esta recarregando!\n");
        } else if (opcao == ACAO_FUGIR && podeFugir == 0) {
            printf("Nao ha como fugir deste combate!\n");
        } else {
            return opcao;
        }
    }
}

void ataqueJogador(Personagem *p, Monstro *m, int especial) {
    int d20, dano, acertou, critico, curado, extra;

    d20 = rolarDado(20);
    printf("  Dado de acerto (d20): %d  (esquiva do inimigo: %d)\n", d20, m->esquiva);

    if (p->classe == CLASSE_ESPECIALISTA && d20 != 20 && d20 < m->esquiva) {
        printf("  [Passiva] Voce errou! Rola o d20 de novo...\n");
        d20 = rolarDado(20);
        printf("  Novo d20: %d\n", d20);
    }

    critico = 0;
    acertou = 0;
    if (d20 == 20) {
        critico = 1;
        acertou = 1;
    } else if (d20 != 1 && d20 >= m->esquiva) {
        acertou = 1;
    }

    if (acertou == 0) {
        if (d20 == 1 && p->classe != CLASSE_ESPECIALISTA) {
            printf("  ERRO CRITICO! Voce se desequilibra e erra feio.\n");
        } else {
            printf("  Voce errou o golpe.\n");
        }
        return;
    }

    dano = calcularDanoJogador(p, especial);
    if (critico == 1) {
        extra = rolarDadoJogador(p, 6);
        printf("  ACERTO CRITICO! +%d de dano extra (1d6).\n", extra);
        dano = dano + extra;
    }
    if (dano < 1) {
        dano = 1;
    }

    danoNoMonstro(m, dano);
    printf("  Voce causa %d de dano em %s!\n", dano, m->nome);

    /* Suga Vida: cura 50% do dano (arredonda para baixo) */
    if (p->sugaCargas > 0) {
        curado = curar(p, dano / 2);
        p->sugaCargas = p->sugaCargas - 1;
        printf("  [Suga Vida] Voce recupera %d de vida. (%d ataque(s) restante(s))\n",
               curado, p->sugaCargas);
    }
}

int tentarFugir(Personagem *p) {
    int limite, d20;

    limite = LIMITE_FUGA;
    if (p->vida * 2 < p->vidaMax) {
        limite = LIMITE_FUGA_FERIDO;
    }

    d20 = rolarDado(20);
    printf("  Tentativa de fuga: d20 = %d (precisa de %d ou mais)\n", d20, limite);
    if (d20 >= limite) {
        return 1;
    }

    if (p->classe == CLASSE_EXPLORADOR) {
        printf("  [Passiva] Voce falhou, mas tenta de novo!\n");
        d20 = rolarDado(20);
        printf("  Nova tentativa: d20 = %d\n", d20);
        if (d20 >= limite) {
            return 1;
        }
    }
    return 0;
}

int testarDesvio(Personagem *p) {
    int d20, esquiva;

    d20 = rolarDado(20);
    esquiva = calcularEsquiva(p);
    printf("  Dado de desvio (d20): %d + esquiva %d = %d (precisa de %d)\n",
           d20, esquiva, d20 + esquiva, LIMITE_DESVIO);

    if (d20 == 20) {
        return DESVIO_CRITICO;
    }
    if (d20 == 1 && p->classe != CLASSE_ESPECIALISTA) {
        return DESVIO_FALHA_CRITICA;
    }
    if (d20 + esquiva >= LIMITE_DESVIO) {
        return DESVIO_OK;
    }
    return DESVIO_FALHA;
}

void turnoMonstro(Personagem *p, Monstro *m, int defendeu) {
    Dano d;
    int dano, extra, resultado, reducao;

    if (m->temEspecial == 1 && m->recargaAtual == 0) {
        d = m->especial;
        m->recargaAtual = m->recargaEspecial + 1;
        printf("\n[%s usa %s!]\n", m->nome, m->nomeEspecial);
        printf("%s\n", m->textoEspecial);
    } else {
        d = m->ataque;
        printf("%s\n", m->textoAtaque);
    }

    resultado = testarDesvio(p);

    if ((resultado == DESVIO_FALHA || resultado == DESVIO_FALHA_CRITICA)
        && p->classe == CLASSE_EXPLORADOR) {
        printf("  [Passiva] Voce falhou no desvio, mas tenta de novo!\n");
        resultado = testarDesvio(p);
    }

    if (resultado == DESVIO_CRITICO) {
        printf("  DESVIO CRITICO! Voce se esquiva e contra-ataca!\n");
        dano = calcularDanoJogador(p, 0);
        danoNoMonstro(m, dano);
        printf("  Seu contra-ataque causa %d de dano em %s!\n", dano, m->nome);
    } else if (resultado == DESVIO_OK) {
        printf("  Voce desvia do ataque!\n");
    } else {
        dano = rolarDano(d);
        if (resultado == DESVIO_FALHA_CRITICA) {
            extra = rolarDano(m->ataque);
            printf("  FALHA CRITICA! %s acerta um golpe extra de %d de dano!\n", m->nome, extra);
            dano = dano + extra;
        }
        if (defendeu == 1) {
            reducao = p->defesa + rolarDadoJogador(p, 6);
            dano = dano - reducao;
            if (dano < 1) {
                dano = 1;
            }
            printf("  Voce se defende e reduz %d de dano!\n", reducao);
        }
        receberDano(p, dano);
        printf("  %s causa %d de dano em voce!\n", m->nome, dano);
    }

    if (m->recargaAtual > 0) {
        m->recargaAtual = m->recargaAtual - 1;
    }
}

int combate(Personagem *p, Monstro *m, int podeFugir) {
    int turno, acao, defendeu;

    turno = 1;
    p->recargaAtual = 0;
    p->sugaCargas = 0;

    printf("\n!!!!!!!!!!!!!!!! COMBATE !!!!!!!!!!!!!!!!\n");
    printf("Um(a) %s aparece!\n%s\n", m->nome, m->descricao);

    while (1) {
        printf("\n=== Turno %d ===", turno);
        mostrarCombate(p, m);
        defendeu = 0;
        acao = escolherAcao(p, podeFugir);

        if (acao == ACAO_FUGIR) {
            if (tentarFugir(p) == 1) {
                printf("Voce conseguiu fugir do combate!\n");
                return COMBATE_FUGA;
            }
            printf("Voce nao conseguiu fugir!\n");
        } else if (acao == ACAO_ATACAR) {
            ataqueJogador(p, m, 0);
        } else if (acao == ACAO_DEFENDER) {
            defendeu = 1;
            printf("Voce assume uma postura defensiva!\n");
        } else {
            printf("Voce usa %s!\n", p->nomeHabilidade);
            if (p->classe == CLASSE_PREDADOR) {
                p->sugaCargas = 3;
                printf("  Seus proximos 3 ataques vao te curar em 50%% do dano causado.\n");
            } else {
                ataqueJogador(p, m, 1);
            }
            p->recargaAtual = p->recargaHabilidade + 1;
        }
        if (p->recargaAtual > 0) {
            p->recargaAtual = p->recargaAtual - 1;
        }

        if (m->vida <= 0) {
            printf("\n%s foi derrotado(a)!\n", m->nome);
            return COMBATE_VITORIA;
        }

        turnoMonstro(p, m, defendeu);

        if (m->vida <= 0) {
            printf("\n%s foi derrotado(a)!\n", m->nome);
            return COMBATE_VITORIA;
        }

        if (p->vida <= 0) {
            if (tentarReviver(p) == 0) {
                return COMBATE_DERROTA;
            }
        }
        turno = turno + 1;
    }
}

/* 3 vazias, 3 combates, 3 tesouros, 2 curas e 1 saida */
void inicializarTabuleiro(int tabuleiro[]) {
    int layout[CASAS_POR_FASE] = {
        CASA_VAZIA,   CASA_COMBATE, CASA_CURA,    CASA_TESOURO,
        CASA_COMBATE, CASA_CURA,    CASA_VAZIA,   CASA_TESOURO,
        CASA_VAZIA,   CASA_COMBATE, CASA_TESOURO, CASA_SAIDA
    };
    int i;
    for (i = 0; i < CASAS_POR_FASE; i++) {
        tabuleiro[i] = layout[i];
    }
}

char simboloCasa(int tipo) {
    switch (tipo) {
        case CASA_VAZIA:   return '.';
        case CASA_COMBATE: return 'C';
        case CASA_TESOURO: return 'T';
        case CASA_CURA:    return '+';
        default:           return 'S';
    }
}

void mostrarTabuleiro(int tabuleiro[], int posicao) {
    int i;

    printf("\n Mapa: ");
    for (i = 0; i < CASAS_POR_FASE; i++) {
        if (i + 1 == posicao) {
            printf("[@]");
        } else {
            printf("[%c]", simboloCasa(tabuleiro[i]));
        }
    }
    printf("\n (@ voce | C combate | T tesouro | + cura | . vazia | S saida)\n");
    printf(" Casa atual: %d/%d\n", posicao, CASAS_POR_FASE);
}

void moverJogador(int *posicao, int passos) {
    *posicao = *posicao + passos;
    if (*posicao > CASAS_POR_FASE) {
        *posicao = CASAS_POR_FASE;
    }
}

int menuTabuleiro() {
    printf("\nO que deseja fazer?\n");
    printf("  [1] Rolar dado\n  [2] Ver status\n  [3] Abrir bolsa\n");
    return lerOpcao("Escolha: ", 1, 3);
}

int executarEvento(Personagem *p, Fase *f, int numeroFase, int tipo, int indiceVazia) {
    Monstro m;
    int qtd, resultado, d20, cura;

    if (tipo == CASA_VAZIA) {
        printf("\n[Casa vazia]\n");
        mostrarHistoriaCasaVazia(numeroFase, indiceVazia);
        pausar();

    } else if (tipo == CASA_COMBATE) {
        /* na fase 3 so os 2 primeiros aparecem; o Palhaco e o chefe final */
        qtd = MONSTROS_POR_FASE;
        if (numeroFase == NUM_FASES) {
            qtd = 2;
        }
        /* copia do monstro: o original nao perde vida */
        m = f->monstros[rand() % qtd];
        resultado = combate(p, &m, 1);
        if (resultado == COMBATE_DERROTA) {
            return 0;
        }
        if (resultado == COMBATE_VITORIA) {
            ganharXP(p, m.xpRecompensa);
        }
        pausar();

    } else if (tipo == CASA_TESOURO) {
        printf("\n[Casa de tesouro] Voce encontra um bau trancado!\n");
        d20 = rolarDado(20);
        printf("  Rolagem para abrir o bau (d20): %d\n", d20);
        printf("Voce abre o bau e encontra suprimentos do circo antigo.\n");
        ganharXP(p, XP_TESOURO);
        if (d20 >= LIMITE_POCAO_BAU) {
            p->pocoes = p->pocoes + 1;
            printf("Dentro dele, tambem havia uma POCAO DE CURA! (Pocoes: %d)\n", p->pocoes);
        }
        pausar();

    } else if (tipo == CASA_CURA) {
        printf("\n[Casa de cura] Um canto seguro. Voce descansa e trata seus ferimentos.\n");
        cura = curar(p, p->vidaMax / 2);
        printf("Voce recupera %d de vida. (Vida: %d/%d)\n", cura, p->vida, p->vidaMax);
        pausar();

    } else {
        if (numeroFase == NUM_FASES) {
            m = f->monstros[2];
            printf("\n[Arena] O Palhaco te espera no centro da arena.\n");
            pausar();
            if (combate(p, &m, 0) == COMBATE_DERROTA) {
                return 0;
            }
            ganharXP(p, m.xpRecompensa);
            revelacaoDoPalhaco();
        } else {
            printf("\n[Saida]\n");
            mostrarSaidaFase(numeroFase);
        }
        pausar();
    }
    return 1;
}

int jogarFase(Personagem *p, Fase *f, int numeroFase) {
    int tabuleiro[CASAS_POR_FASE];
    int posicao, acao, d6, i, indiceVazia;

    posicao = 0;
    inicializarTabuleiro(tabuleiro);

    printf("\n==============================================================\n");
    printf(" FASE %d - %s\n", numeroFase, f->nome);
    printf("==============================================================\n");
    printf("MISSAO: %s\n\n", f->missao);
    mostrarIntroducaoFase(numeroFase);
    pausar();

    while (posicao < CASAS_POR_FASE) {
        mostrarTabuleiro(tabuleiro, posicao);
        acao = menuTabuleiro();

        if (acao == MENU_STATUS) {
            mostrarStatus(p);
        } else if (acao == MENU_BOLSA) {
            abrirBolsa(p);
        } else {
            d6 = rolarDado(6);
            printf("\nVoce rolou o d6: %d\n", d6);
            moverJogador(&posicao, d6);
            printf("Voce avanca para a casa %d.\n", posicao);

            indiceVazia = 0;
            for (i = 0; i < posicao - 1; i++) {
                if (tabuleiro[i] == CASA_VAZIA) {
                    indiceVazia = indiceVazia + 1;
                }
            }

            if (executarEvento(p, f, numeroFase, tabuleiro[posicao - 1], indiceVazia) == 0) {
                return 0;
            }
        }
    }

    printf("\n>>> MISSAO CUMPRIDA: %s <<<\n", f->nome);
    return 1;
}

int main() {
    Personagem heroi;
    Fase fases[NUM_FASES];
    int i;

    srand(time(NULL));

    inicializarFases(fases);
    introducao();
    escolherPersonagem(&heroi);

    for (i = 0; i < NUM_FASES; i++) {
        if (jogarFase(&heroi, &fases[i], i + 1) == 0) {
            printf("\n==============================================================\n");
            printf("                        GAME OVER\n");
            printf("==============================================================\n");
            printf("%s caiu em %s. A Noite Magenta cobra mais uma vida...\n",
                   heroi.nome, fases[i].nome);
            return 0;
        }
    }

    printf("\n==============================================================\n");
    printf("                         VITORIA!\n");
    printf("==============================================================\n");
    printf("%s completou todas as missoes e descobriu a verdade sobre o circo.\n", heroi.nome);
    mostrarStatus(&heroi);
    return 0;
}
