# Tomada Vital — app da família

Aplicativo Flutter da **Tomada Vital**, solução da Global Solution 2026/2 (CitizenTech) da FIAP.

Quando a energia cai, quem depende de um aparelho ligado na tomada fica só com a bateria dele. A Tomada Vital é um dispositivo na tomada que percebe a queda, lê a bateria e avisa a central pela rede celular. A central monta uma fila em que **quem tem menos tempo de bateria é atendido primeiro**. Este app é o lado da **família**: mostra se a casa está sem energia, quantos minutos de bateria restam, a posição na fila e o que a central já fez.

> **Vídeo de demonstração:** [https://youtu.be/cv3VCcZC_rg](https://youtu.be/cv3VCcZC_rg)

---

## O problema e quem é beneficiado

- **Cidadão beneficiado:** pessoas eletrodependentes, que usam em casa concentrador de oxigênio, ventilador, aspirador de secreção ou bomba de infusão, e as famílias que cuidam delas.
- **O problema:** no apagão de 10/12/2025, a Enel admitiu 4,4 milhões de clientes atingidos em São Paulo. No temporal de 11/10/2024, Taboão da Serra ainda tinha 32,7 mil instalações sem energia no 3º dia. A concessionária não sabe quais casas de eletrodependentes estão sem luz, nem quanto tempo cada uma aguenta, e a família fica sem resposta.
- **O que o app entrega:** a situação da casa em tempo real, a urgência calculada a partir da bateria, a fila de atendimento, o histórico do que a central fez e os pontos de apoio com energia de reserva.

### ODS relacionados

| ODS | Relação |
|---|---|
| **3** Saúde e bem-estar | protege a vida de quem depende de aparelho médico em casa |
| **9** Indústria, inovação e infraestrutura | dispositivo conectado e fila de atendimento automática |
| **11** Cidades e comunidades sustentáveis | resposta organizada da cidade em apagões |
| **13** Ação contra a mudança do clima | os apagões longos vêm de eventos climáticos extremos |

---

## Telas

| Login | Validação do login | Minha casa | Meus equipamentos |
|---|---|---|---|
| <img src="docs/prints/01_login.png" width="200"> | <img src="docs/prints/02_login_validacao.png" width="200"> | <img src="docs/prints/03_minha_casa.png" width="200"> | <img src="docs/prints/04_equipamentos.png" width="200"> |

| Detalhe do equipamento | Cadastro com validação | Atendimento | Pontos de apoio |
|---|---|---|---|
| <img src="docs/prints/05_detalhe.png" width="200"> | <img src="docs/prints/06_cadastro_validacao.png" width="200"> | <img src="docs/prints/07_atendimento.png" width="200"> | <img src="docs/prints/08_pontos_de_apoio.png" width="200"> |

| Tela | O que faz |
|---|---|
| **Login** | Formulário com validação dos campos. Usuário ou senha errados abrem um `AlertDialog` |
| **Minha casa** | Situação da energia. Os botões **Simular queda**, **Passar 30 min** e **Energia voltou** mudam o estado da tela com `setState` (minutos, prioridade, cor e posição na fila) |
| **Meus equipamentos** | `ListView` com os aparelhos da família e o status de cada um. O botão **Cadastrar** abre o formulário, e a lista se atualiza na volta |
| **Detalhe do equipamento** | Recebe o equipamento **por parâmetro**. Mostra bateria, minutos restantes, prioridade e posição na fila |
| **Cadastrar equipamento** | `Form` + `GlobalKey<FormState>`: código no formato `TV-000` e sem repetir, tipo (`DropdownButtonFormField`) e autonomia de 0 a 1440 min. Ao salvar, mostra um `SnackBar` |
| **Atendimento** | Recebe a ocorrência **por parâmetro**. Linha do tempo com o que a central já fez e o que falta |
| **Pontos de apoio** | `GridView` com locais que têm energia de reserva. Os favoritos e o contador mudam com `setState` |

---

## Regra de prioridade

A mesma regra é usada no app, no banco de dados, na API e no dispositivo:

- **Minutos restantes** = autonomia do aparelho × bateria (%) ÷ 100 − minutos desde a queda
- **CRÍTICA:** até 120 min, ou aparelho sem bateria · **ALTA:** 121 a 360 min · **MODERADA:** acima de 360 min
- **Fila:** menos minutos primeiro; no empate, quem perdeu a energia antes

---

## Arquitetura

```
lib/
├── main.dart · app.dart     # entrada, tema e rota inicial
├── models/                  # Patient, Equipment, Outage, ResponseStep, SupportPoint e enums
├── data/                    # dados de exemplo (os mesmos do banco Oracle do projeto)
├── repositories/            # acesso aos dados: busca, cadastro e fila de atendimento
├── services/                # regras de negócio: prioridade, validações, login, quedas, linha do tempo
├── theme/                   # cores, espaçamentos e ThemeData únicos do app
├── utils/                   # textos dos enums em português e formatação de horas e durações
├── routes/                  # nomes das rotas
├── widgets/                 # componentes reutilizáveis (cards, botões, campos, badges, timeline)
└── screens/                 # uma pasta por tela
```

Decisões de projeto:

- **Regras de negócio fora da interface.** A prioridade, a fila, as validações e a simulação de queda ficam em `services/` e `repositories/`. As telas só montam a interface, chamam essas classes e atualizam o estado.
- **Componentes reutilizáveis.** Cards, botões, campos de texto, badges de prioridade e itens da linha do tempo ficam em `widgets/`, e toda tela usa os mesmos.
- **Identidade visual única.** Todas as cores e espaçamentos vêm de `theme/`.
- **Código em inglês, interface em português.** Classes, métodos e variáveis estão em inglês. Os textos exibidos ao usuário ficam em português (`utils/labels.dart`).
- **Sem pacotes externos.** Só o que o Flutter oferece: `Navigator`, `StatefulWidget` + `setState`, `Form` e os widgets do Material.
- **Dados de exemplo.** Esta versão não consome API: os dados ficam em `data/mock_data.dart`, com os mesmos pacientes, equipamentos e ocorrências do banco de dados do projeto.

### Widgets da disciplina usados

`Scaffold`, `AppBar`, `Column`, `Row`, `Container`, `Padding`, `SizedBox`, `Card`, `Image`, `Icon`, `ListView`, `GridView`, `Form`, `TextFormField`, `DropdownButtonFormField`, `FloatingActionButton`, `SnackBar` e `AlertDialog`.

---

## Como executar

Pré-requisitos: Flutter instalado (`flutter doctor` sem erro em Flutter e Android toolchain) e um emulador Android ou um celular com depuração USB.

```bash
git clone https://github.com/ryanmazali/tomada-vital-mobile.git
cd tomada-vital-mobile
flutter pub get
flutter run
```

**Usuário de teste:** `familia1` · **senha:** `123456`

Roteiro rápido: entre com o usuário de teste. Em **Minha casa**, use **Passar 30 min** e **Energia voltou**, e depois **Simular queda de energia**. Abra **Equipamentos**, toque no TV-001 e em **Ver atendimento**. Volte, toque em **Cadastrar** e tente salvar com o código `TV-12` para ver a validação.

---

## Integrantes

| Nome | RM | Turma |
|---|---|---|
| Diego Paulino | RM566841 | 2TDSPR |
| Guilherme Dabul | RM559901 | 2TDSPR |
| Renan Lima | RM568321 | 2TDSPR |
| Ryan Mazali | RM567168 | 2TDSPR |
| Thiago Alexandre | RM567486 | 2TDSPR |

---

## Uso de IA generativa

| Ferramenta | Para quê |
|---|---|
| Claude (Anthropic) | Apoio na organização da arquitetura em camadas, na escrita e revisão do código das telas e componentes, na prévia visual das telas e na redação deste README |

Todo o código foi revisado e testado pelo grupo no emulador Android.
