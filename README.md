Mirabilia
Aluno: Magdiel Toscano de Azevedo
Matricula: 20260042270
Disciplina: Programacao | Padrao de C: C99
RPG de tabuleiro em modo texto, feito em C, que roda inteiramente no terminal.
Historia
Ha 200 anos, a cidade de Mirabilia foi atingida pela Noite Magenta, um fenomeno de origem desconhecida: um ceu rosa visivel apenas sobre a cidade. Junto com ele surgiram as anomalias, criaturas estranhas, e muitos humanos enlouqueceram ou se transformaram nelas. Alguns humanos, porem, conseguiram conter a energia anomala e enfrentaram o caos: os primeiros Espers.
Hoje, a cidade esta isolada do resto do mundo e o DCA (Departamento de Combate a Anomalias) e a principal forca de seguranca. Espers sao obrigados por lei a servir ao DCA, e quem nao despertou tambem pode servir com apoio tecnologico.
Voce e um agente recem-treinado do DCA em sua primeira missao na Zona de Quarentena. Para cumprir as missoes, atravessara tres locais: a propria zona de quarentena, os Tuneis do Riso no subsolo e, por fim, O Circo, onde o Palhaco revela a verdade sobre o que aconteceu ali.
Objetivo final: cumprir a missao das 3 fases e derrotar o Palhaco.
Personagens
Classe	Vida	Forca	Agilidade	Inteligencia	Defesa	Ataque basico	Habilidade especial	Passiva
Explorador	20	15	25	10	2	4 + 1d6	Apunhalar: 4 + 2d6 (recarga 3 turnos)	Se falhar ao desviar ou fugir, tenta mais uma vez
Predador	35	25	15	10	3	3 + 1d6	Suga Vida: os proximos 3 ataques curam 50% do dano causado, arredondando para baixo (recarga 6 turnos)	Uma tentativa extra para reviver
Especialista	15	10	15	25	2	5 + 1d6	Energia de Anomalia: 5 + 1d8 (recarga 2 turnos)	Nunca erra criticamente e rola de novo um d20 que errou ou um dado que deu 1
Equipamentos (descritivos): Explorador - Faca de combate do DCA e Colete tatico leve; Predador - Manoplas de contencao e Armadura reforcada do DCA; Especialista - Emissor de energia anomala e Jaleco-uniforme com nucleo anomalo.
Fases e monstros
Fase	Missao	Monstros (vida / esquiva / XP / ataque)
1 - Zona de Quarentena	Atravessar a zona e achar o bueiro para o subsolo	Leao Marinho (12/3/2/1d4), Cachorro (15/5/3/1+1d4), Leao (18/6/4/2+1d6)
2 - Tuneis do Riso	Atravessar os tuneis ate a lona do circo	Trapezista (14/10/5/1d4), Monociclista (14/11/6/1d6), Dancarina (14/12/7/1d8)
3 - O Circo	Chegar a arena e enfrentar o Palhaco	Contorcionista (14/13/8/1d6), Magico (15/10/9/1+1d4; Bombas da Cartola 1+2d4, recarga 4), Palhaco (20/12/10/2+1d6; Torta Toxica 1+2d6, recarga 3)
Na fase 3, as casas de combate sorteiam entre Contorcionista e Magico. O Palhaco e o chefe da casa final e nao permite fuga.
Dinamica do jogo
Tabuleiro: cada fase tem 12 casas: 3 vazias (contam um pouco da historia do circo), 3 de combate, 3 de tesouro, 2 de cura (sempre depois de um combate) e 1 de saida (missao). A cada turno no tabuleiro o jogador escolhe: rolar o dado, ver status ou abrir a bolsa.
Movimentacao: d6. O heroi avanca esse numero de casas e o evento acontece na casa onde parou (casas puladas nao disparam evento). Nao passa da casa 12.
Eventos:
Combate: monstro sorteado da fase.
Tesouro: rola um d20; sempre rende 3 XP e, com 15 ou mais, tambem uma pocao de cura.
Cura: recupera 50% da vida maxima.
Pocao: recupera 10 de vida (usada na bolsa).
Saida: cumpre a missao da fase. Na fase 3 e a luta contra o Palhaco.
Combate por turnos (acoes: Atacar, Habilidade especial, Defender, Fugir):
Acerto: o jogador rola d20; acerta se `d20 >= esquiva do monstro`. 20 = acerto critico (+1d6 de dano). 1 = erro critico.
Dano do jogador: `base + dados + forca/5` (forca so no ataque basico) `+ inteligencia/5` (so no nivel 2+). Minimo 1.
Ataque do monstro: o jogador rola um d20 de desvio e desvia se `d20 + esquiva >= 15`. 20 = desvio critico (contra-ataque basico automatico). 1 = falha critica (leva um ataque basico extra do monstro).
Esquiva do jogador: `agilidade/10` no nivel 1 e `agilidade/5` do nivel 2 em diante. A esquiva evita o golpe.
Defender: o jogador gasta o turno sem atacar. Se o monstro acertar, o dano diminui em `defesa + 1d6` (dano minimo 1). A defesa reduz o dano, e nao muda com o nivel.
Fuga: d20 `>= 10`; se a vida estiver abaixo da metade, `>= 14`. Fugir nao rende recompensa. O monstro ataca se a fuga falhar.
Recarga: habilidades com recarga de N turnos ficam indisponiveis nos N turnos seguintes ao uso. A recarga zera ao fim de cada combate.
Evolucao:
XP necessario por nivel: 3, 5, 7, 9, 11, 13 (nivel maximo 7). Ao subir, o XP zera.
Ao subir de nivel, o atributo principal ganha +5 e um dos outros dois atributos, sorteado, ganha +2 (principal: Explorador agilidade; Predador forca; Especialista inteligencia).
Desperto (nivel 2+): +1 de vida maxima a cada 5 de vida maxima (a cada nivel), esquiva `agilidade/5` e +1 de dano a cada 5 de inteligencia.
Derrota e vitoria: ao chegar a 0 de vida o heroi desmaia e tem 3 tentativas de reviver (4 para o Predador), cada uma com d20 `>= 10`; reviver devolve 25% da vida maxima. Se todas falharem, game over. O jogador vence ao cumprir a missao das 3 fases.
Como compilar e jogar
Pre-requisito: `gcc`.
```bash
# Linux / macOS / Windows (MinGW)
gcc -std=c99 -Wall -Wextra -o jogo rpg.c
./jogo        # no Windows: jogo.exe
```
(Se mover o arquivo para `src/rpg.c`, use `gcc -std=c99 -Wall -Wextra -o jogo src/rpg.c`.)
Estrutura do codigo
enums: `ClassePersonagem`, `TipoCasa`, `AcaoCombate`, `AcaoTabuleiro`, `ResultadoCombate`, `ResultadoDesvio`.
structs: `Dano` (base + dados), `Personagem`, `Monstro`, `Fase`.
ponteiros: `receberDano`, `danoNoMonstro`, `curar`, `ganharXP`, `subirDeNivel`, `tentarReviver`, `moverJogador(int \*posicao, ...)`, `combate(Personagem \*, Monstro \*, ...)`.
funcoes principais: `introducao`, `escolherPersonagem`, `inicializarFases`, `jogarFase`, `executarEvento`, `combate`, `ataqueJogador`, `turnoMonstro`, `lerOpcao` (entrada validada com `scanf`).

Video de demonstracao
https://www.youtube.com/watch?v=IiHpE9rWncc (Video gravado no terminal do VSCODE pois o Windows estava bloqueando meu GCC)
