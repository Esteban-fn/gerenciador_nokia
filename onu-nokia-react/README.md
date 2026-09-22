# Gerenciador Nokia ONU

Aplicacao web para ajudar o suporte tecnico a montar comandos de configuracao para ONUs Nokia. O usuario informa a posicao da ONU e os dados desejados, e o sistema gera o comando e o copia para a area de transferencia.

> **Guia para aprender e evoluir o projeto:** consulte [GUIA-DE-APRENDIZADO.md](GUIA-DE-APRENDIZADO.md). O conteúdo original deste README foi mantido.

> Este projeto e um frontend. Ele nao se conecta diretamente a uma OLT ou ONU: atualmente ele gera textos de comandos para que o tecnico possa cola-los em outra ferramenta.

## 1. Explicacao simples

Imagine o projeto como uma casa:

- `src/main.jsx` abre a casa.
- `src/App.jsx` e o corredor principal e decide qual tela sera mostrada.
- `src/components/` contem os comodos, ou seja, as telas e partes visuais.
- `src/services/` contem as receitas que montam os comandos Nokia.
- `src/utils/` contem ferramentas pequenas, como validacoes e copia de texto.
- `src/context/` guarda informacoes compartilhadas entre as telas.
- `src/hooks/` contem comportamentos reutilizaveis, como alertas e atalhos.
- `src/*.css` define as cores, tamanhos, espacamentos e responsividade.

Quando voce clica em uma opcao do menu, o React troca o componente visivel sem recarregar a pagina inteira.

## 2. Objetivo do sistema

O sistema organiza tarefas comuns de atendimento e configuracao de ONUs Nokia, como:

- pesquisar cliente;
- provisionar cliente;
- configurar Wi-Fi;
- alterar VLAN e PPPoE;
- alterar senha web;
- configurar telefone;
- gerar comandos de bridge;
- conferir dados de uma caixa;
- executar outras operacoes disponiveis no menu.

As telas normalmente recebem a posicao da ONU (slot, porta PON e indice), validam os campos, geram um comando, copiam o resultado e exibem uma mensagem.

## 3. Tecnologias usadas

### React

React e a biblioteca usada para construir a interface com pecas reutilizaveis. Essas pecas sao chamadas de componentes e ficam principalmente em arquivos `.jsx`.

Exemplo:

```jsx
function Saudacao() {
   return <h1>Ola!</h1>;
}
```

### JavaScript e JSX

JavaScript contem a logica do programa. JSX parece HTML, mas fica dentro do JavaScript e permite criar a tela.

### Vite

Vite e a ferramenta que inicia o servidor de desenvolvimento e prepara o projeto para publicacao. Ele oferece atualizacao quase imediata enquanto voce edita os arquivos.

### CSS

CSS controla a aparencia. O projeto usa `App.css`, `index.css` e alguns arquivos CSS especificos de componentes.

### Bibliotecas

- `react-icons`: icones do menu e da interface.
- `sweetalert2`: janelas e mensagens de sucesso, erro e confirmacao.
- `gh-pages`: publica a pasta final no GitHub Pages.

## 4. Preparando o ambiente

Voce precisa ter Node.js instalado. Depois, abra o terminal na pasta `onu-nokia-react`:

```bash
cd onu-nokia-react
npm install
npm run dev
```

Abra o endereco mostrado pelo terminal, normalmente:

```text
http://localhost:5173
```

Para parar o servidor, pressione `Ctrl+C`.

### Comandos importantes

```bash
npm run dev       # inicia o desenvolvimento
npm run build     # cria a versao final em dist/
npm run preview   # visualiza a versao compilada
npm run lint      # procura problemas de codigo
```

O `package.json` importante e o que fica dentro de `onu-nokia-react`. O `package.json` da pasta acima possui dependencias auxiliares do repositorio.

## 5. Mapa dos arquivos

| Caminho | Funcao |
| --- | --- |
| `src/main.jsx` | Ponto inicial da aplicacao |
| `src/App.jsx` | Layout, menu e selecao da tela ativa |
| `src/config/menuItems.js` | Lista de itens do menu lateral |
| `src/context/AppContext.jsx` | Estado compartilhado, como posicao e tela atual |
| `src/components/` | Telas e componentes visuais |
| `src/services/` | Funcoes que montam comandos Nokia |
| `src/utils/validation.js` | Validacoes, copia e listas reutilizaveis |
| `src/hooks/` | Comportamentos reutilizaveis |
| `src/App.css` | Estilos principais da aplicacao |
| `src/index.css` | Estilos globais |
| `vite.config.js` | Configuracao do Vite |
| `package.json` | Scripts e bibliotecas |
| `public/` | Arquivos publicos que nao passam pelo processamento do React |
| `dist/` | Resultado do build; nao e o local normal de edicao |

## 6. Como uma tela funciona

Considere a tela de Wi-Fi:

1. `ConfiguracaoWifi.jsx` desenha os campos e botoes.
2. `useState` guarda o nome e a senha digitados.
3. O componente recebe `posicaoData` com slot, PON e indice.
4. `validation.js` verifica se os dados sao validos.
5. `wifiService.js` monta o texto do comando.
6. `copyToClipboard` copia o texto.
7. `useSweetAlert.js` mostra o resultado para o usuario.

O sistema nao executa o comando na OLT. Ele apenas prepara e copia o texto.

## 7. Como alterar algo existente

### Alterar um texto de botao

Procure o texto dentro do componente, por exemplo em `src/components/ConfiguracaoWifi.jsx`, e altere somente o texto:

```jsx
ALTERAR APENAS NOME
```

### Alterar cores ou tamanho

Procure a classe CSS usada pelo elemento em `App.css` ou no CSS do componente. Nao altere `dist/`: ele sera recriado pelo build.

### Alterar um comando

Abra o servico correspondente em `src/services/`. Os comandos usam template strings, com crases, para inserir valores:

```js
return `Comando com ${data.inputSlot} e ${data.inputIndex}`;
```

Tenha cuidado com aspas, ponto e virgula e com a ordem dos parametros. Uma pequena mudanca pode gerar um comando invalido para a rede.

## 8. Como adicionar uma nova opcao

Suponha que voce queira criar uma tela chamada `Reiniciar ONU`.

### Passo 1: criar o servico

Crie `src/services/reiniciarService.js`:

```js
export function gerarComandoReinicio(data) {
   const { inputSlot, inputGpon, inputIndex } = data;

   return `COMANDO-DE-REINICIO-${inputSlot}-${inputGpon}-${inputIndex};`;
}
```

Substitua o texto pelo comando real validado pela equipe de rede.

### Passo 2: criar a tela

Crie `src/components/ReiniciarOnu.jsx`:

```jsx
import { gerarComandoReinicio } from '../services/reiniciarService';
import { copyToClipboard } from '../utils/validation';

function ReiniciarOnu({ posicaoData }) {
   const handleReiniciar = async () => {
      const comando = gerarComandoReinicio(posicaoData);
      await copyToClipboard(comando);
   };

   return (
      <section className="card">
         <div className="card-header">
            <h3>REINICIAR ONU</h3>
         </div>
         <button type="button" className="btn btn-primary" onClick={handleReiniciar}>
            GERAR COMANDO
         </button>
      </section>
   );
}

export default ReiniciarOnu;
```

### Passo 3: cadastrar no menu

Em `src/config/menuItems.js`, importe um icone e adicione um item:

```js
import { FiRefreshCw } from 'react-icons/fi';

// dentro de menuItems
{ id: 'reiniciar', label: 'Reiniciar ONU', icon: FiRefreshCw },
```

### Passo 4: conectar a tela ao `App.jsx`

Em `src/App.jsx`, importe o componente:

```jsx
import ReiniciarOnu from './components/ReiniciarOnu';
```

Depois, adicione o caso no `switch` de `renderActiveComponent`:

```jsx
case 'reiniciar':
   return <ReiniciarOnu posicaoData={posicaoData} />;
```

Agora o `id` do menu e o `case` precisam ser iguais. Se um se chamar `reiniciar` e o outro `restart`, a opcao nao abrira.

### Passo 5: testar

```bash
npm run lint
npm run build
```

Depois, abra a tela no navegador e teste campos vazios, valores validos e o comando copiado.

## 9. Conceitos de codigo para aprender

### Componente

Uma funcao que retorna uma parte da tela:

```jsx
function Titulo() {
   return <h2>Minha tela</h2>;
}
```

### Propriedade (`props`)

E uma informacao enviada de um componente para outro:

```jsx
function Tela({ nome }) {
   return <p>{nome}</p>;
}
```

### Estado (`useState`)

E uma informacao que pode mudar enquanto a pessoa usa a tela:

```jsx
const [nome, setNome] = useState('');
```

### Evento

E uma acao do usuario, como clicar ou digitar:

```jsx
<button onClick={handleSalvar}>Salvar</button>
```

### Funcao

E uma receita que recebe dados e devolve um resultado. Os arquivos em `services/` usam funcoes para montar comandos sem desenhar a tela.

### Import e export

Permitem reutilizar codigo entre arquivos:

```js
// arquivo origem
export function somar(a, b) {
   return a + b;
}

// outro arquivo
import { somar } from './arquivo';
```

## 10. Fluxo recomendado para qualquer mudanca

1. Descreva a mudanca em uma frase.
2. Localize a tela em `src/components/`.
3. Se houver comando, localize o servico em `src/services/`.
4. Faça uma alteracao pequena.
5. Salve e veja o resultado no navegador.
6. Rode `npm run lint`.
7. Rode `npm run build`.
8. So depois publique ou compartilhe a mudanca.

## 11. Problemas comuns

### `npm run dev` nao funciona

Confirme que o terminal esta em `onu-nokia-react` e execute `npm install`.

### A nova opcao aparece no menu, mas nao abre

Confira se o `id` em `menuItems.js` e o `case` em `App.jsx` sao exatamente iguais.

### O estilo nao mudou

Confira o nome da classe no JSX e no CSS. Verifique tambem se editou o arquivo em `src/`, e nao a pasta `dist/`.

### O comando nao e copiado

A copia depende da permissao do navegador e normalmente funciona em `localhost` ou em uma pagina HTTPS. Veja o console do navegador para detalhes.

### O build falhou

Leia a primeira mensagem de erro, corrija aquele ponto e rode o comando novamente. Geralmente o erro esta relacionado a import com nome errado, virgula faltando ou componente sem `export default`.

## 12. Publicacao

O projeto possui configuracao para GitHub Pages:

```bash
npm run build
npm run deploy
```

Antes de publicar, teste localmente e confirme que os comandos gerados estao corretos. A pasta `dist/` e criada automaticamente pelo Vite.

## 13. Roteiro de aprendizagem

Uma ordem tranquila para aprender este projeto:

1. HTML basico: elementos, formularios, botoes e atributos.
2. CSS basico: classes, flexbox, grid, cores e responsividade.
3. JavaScript: variaveis, funcoes, objetos, arrays e template strings.
4. React: componentes, props, estado e eventos.
5. Projeto: imports, servicos, hooks e contexto.
6. Qualidade: validacao, lint, build e testes manuais.
7. Git: commits pequenos, branches e diferenca entre seu fork e o repositorio original.

O melhor exercicio inicial e copiar uma tela simples, trocar o titulo e adicionar um campo. Depois, crie uma opcao que gere um comando de teste. Assim voce aprende a tela e a logica separadamente.

## 14. Regra de ouro

Edite sempre dentro de `src/`, faça uma mudanca por vez e valide o resultado. Quando nao souber onde mexer, procure primeiro o texto que aparece na tela usando a busca do VS Code (`Ctrl+Shift+F`).
# 🔮 Gerenciador Nokia ONU | Tech Cyberpunk# 🔮 Gerenciador Nokia ONU | Tech Cyberpunk

<div align="center"><div align="center">

![Nokia ONU](https://img.shields.io/badge/Nokia-Gerenciador_ONU-98c913?style=for-the-badge&logo=nokia&logoColor=white)![Nokia ONU](https://img.shields.io/badge/Nokia-Gerenciador_ONU-98c913?style=for-the-badge&logo=nokia&logoColor=white)

![React](https://img.shields.io/badge/React-18.3.1-61DAFB?style=for-the-badge&logo=react&logoColor=black)![React](https://img.shields.io/badge/React-18.3.1-61DAFB?style=for-the-badge&logo=react&logoColor=black)

![Vite](https://img.shields.io/badge/Vite-7.1.7-646CFF?style=for-the-badge&logo=vite&logoColor=white)![Vite](https://img.shields.io/badge/Vite-7.1.7-646CFF?style=for-the-badge&logo=vite&logoColor=white)

![GitHub Pages](https://img.shields.io/badge/GitHub-Pages-222222?style=for-the-badge&logo=github&logoColor=white)![GitHub Pages](https://img.shields.io/badge/GitHub-Pages-222222?style=for-the-badge&logo=github&logoColor=white)

_Sistema futurista de gerenciamento com tema duplo para dispositivos Nokia ONU\_\_Sistema futurista de gerenciamento com tema duplo para dispositivos Nokia ONU_

[🚀 Demo ao Vivo](https://MatheusAraujo86.github.io/obscure-xylophone/) • [📖 Recursos](#-recursos) • [🛠️ Instalação](#-primeiros-passos) • [🎨 Temas](#-sistema-de-tema-duplo)[🚀 Demo ao Vivo](https://MatheusAraujo86.github.io/obscure-xylophone/) • [📖 Recursos](#-recursos) • [🛠️ Instalação](#-primeiros-passos) • [🎨 Temas](#-sistema-de-tema-duplo)

---

</div></div>

## ✨ Visão Geral## ✨ Visão Geral

**Gerenciador Nokia ONU** é uma aplicação web de última geração projetada para gerenciar dispositivos Nokia Optical Network Unit (ONU) com estética visual duplamente impressionante. Construído com tecnologia React moderna, fornece aos administradores de rede ferramentas poderosas para configurar, monitorar e solucionar problemas de equipamentos Nokia ONU através de uma interface intuitiva que se adapta entre o modo escuro cyberpunk e o modo claro moderno.**Gerenciador Nokia ONU** é uma aplicação web de última geração projetada para gerenciar dispositivos Nokia Optical Network Unit (ONU) com estética visual duplamente impressionante. Construído com tecnologia React moderna, fornece aos administradores de rede ferramentas poderosas para configurar, monitorar e solucionar problemas de equipamentos Nokia ONU através de uma interface intuitiva que se adapta entre o modo escuro cyberpunk e o modo claro moderno.

## 🎨 Sistema de Tema Duplo## 🎨 Sistema de Tema Duplo## 🎨 Sistema de Tema Duplo

### 🌙 **Modo Escuro - Estética Cyberpunk**### 🌙 **Modo Escuro - Estética Cyberpunk**### 🌙 **Modo Escuro - Estética Cyberpunk**

- **Cores Neon**: Acentos em Ciano (#00FFFF) e Magenta (#FF00FF)- **Cores Neon**: Acentos em Ciano (#00FFFF) e Magenta (#FF00FF)- **Cores Neon**: Acentos em Ciano (#00FFFF) e Magenta (#FF00FF)

- **Fundos Escuros**: Pretos profundos e azuis escuros

- **Efeitos Brilhantes**: Iluminação neon e efeitos de sombra- **Fundos Escuros**: Pretos profundos e azuis escuros- **Fundos Escuros**: Pretos profundos e azuis escuros

- **Tipografia**: Fontes futuristas Orbitron e Roboto Mono

- **Efeitos Brilhantes**: Iluminação neon e efeitos de sombra- **Efeitos Brilhantes**: Iluminação neon e efeitos de sombra

### ☀️ **Modo Claro - Verde Moderno**

- **Tipografia**: Fontes futuristas Orbitron e Roboto Mono- **Tipografia**: Fontes futuristas Orbitron e Roboto Mono

- **Cor Primária**: Verde Lima (#98c913)

- **Fundos Limpos**: Cinzas claros e brancos### ☀️ **Modo Claro - Verde Moderno**### ☀️ **Modo Claro - Verde Moderno**

- **Efeitos Sutis**: Efeitos de brilho verde e sombras

- **Visual Profissional**: Design limpo, moderno e energeticamente eficiente- **Cor Primária**: Verde Lima (#98c913)- **Cor Primária**: Verde Lima (#98c913)

### 🔄 **Alternador de Tema**- **Fundos Limpos**: Cinzas claros e brancos- **Fundos Limpos**: Cinzas claros e brancos

- **Botão Flutuante**: Alternador de tema de fácil acesso- **Efeitos Sutis**: Efeitos de brilho verde e sombras- **Efeitos Sutis**: Efeitos de brilho verde e sombras

- **Troca Instantânea**: Transição de tema em tempo real

- **Persistência**: Preferência de tema salva no localStorage- **Visual Profissional**: Design limpo, moderno e energeticamente eficiente- **Visual Profissional**: Design limpo, moderno e energeticamente eficiente

- **Universal**: Afeta toda a aplicação incluindo o rodapé

### 🔄 **Alternador de Tema**### 🔄 **Alternador de Tema**

## 🚀 Recursos

- **Botão Flutuante**: Alternador de tema de fácil acesso- **Botão Flutuante**: Alternador de tema de fácil acesso

### 🔍 **Gerenciamento de Clientes**

- **Troca Instantânea**: Transição de tema em tempo real- **Troca Instantânea**: Transição de tema em tempo real

- **Busca Avançada de Clientes**: Localiza clientes por vários parâmetros com ícones geométricos cyberpunk

- **Posicionamento de Clientes**: Rastreamento em tempo real de slot/GPON/índice em layout lado a lado- **Persistência**: Preferência de tema salva no localStorage- **Persistência**: Preferência de tema salva no localStorage

- **Dados Abrangentes do Cliente**: Visualiza informações detalhadas do cliente com cores adaptáveis ao tema

- **Universal**: Afeta toda a aplicação incluindo o rodapé- **Universal**: Afeta toda a aplicação incluindo o rodapéanager | Cyberpunk Tech

### ⚙️ **Configuração de Dispositivos**

## 🚀 Recursos<div align="center">

- **Provisionamento/Desprovisionamento de Clientes**: Gerenciamento completo do ciclo de vida com UI aprimorada

- **Configuração de Wi-Fi**: Gerencia configurações e credenciais de rede sem fio### 🔍 **Gerenciamento de Clientes**![Nokia ONU](https://img.shields.io/badge/Nokia-ONU_Manager-98c913?style=for-the-badge&logo=nokia&logoColor=white)

- **Configuração de Telefone**: Configuração VoIP e linha fixa com formulários melhorados

- **Gerenciamento VLAN/PPPoE**: Configuração de camada de rede![React](https://img.shields.io/badge/React-18.3.1-61DAFB?style=for-the-badge&logo=react&logoColor=black)

- **Controle de Acesso Web**: Senha de administrador e gerenciamento de acesso

- **Bridge ONT Nokia**: Verificação MAC integrada e geração de comandos- **Busca Avançada de Clientes**: Localiza clientes por vários parâmetros com ícones geométricos cyberpunk![Vite](https://img.shields.io/badge/Vite-7.1.7-646CFF?style=for-the-badge&logo=vite&logoColor=white)

### 📊 **Ferramentas Avançadas**- **Posicionamento de Clientes**: Rastreamento em tempo real de slot/GPON/índice em layout lado a lado![GitHub Pages](https://img.shields.io/badge/GitHub-Pages-222222?style=for-the-badge&logo=github&logoColor=white)

- **Sistema de Conferência de Caixa**: Compara tabelas antes/depois com estilo de botão unificado- **Dados Abrangentes do Cliente**: Visualiza informações detalhadas do cliente com cores adaptáveis ao tema

- **Detecção de Diferenças**: Destaque automático de mudanças em ambos os temas

- **Rastreamento de Clientes Offline**: Monitora dispositivos offline com UI consistente*A futuristic dual-theme management system for Nokia ONU devices*

- **Exportação/Importação de Dados**: Backup e restauração de configurações

- **Geração de Comandos**: Copia comandos de rede para a área de transferência com notificações### ⚙️ **Configuração de Dispositivos**

### 🎯 **Operações Adicionais**[🚀 Live Demo](https://MatheusAraujo86.github.io/obscure-xylophone/) • [📖 Features](#-features) • [🛠️ Installation](#-installation) • [� Themes](#-dual-theme-system)

- **Busca PON**: Exploração da topologia de rede

- **Monitoramento de Alarmes**: Rastreamento de alertas do sistema- **Provisionamento/Desprovisionamento de Clientes**: Gerenciamento completo do ciclo de vida com UI aprimorada

- **Reinício da ONU**: Funcionalidade de reinicialização remota do dispositivo

- **Solicitações de Provisionamento**: Manipula registros de novos dispositivos- **Configuração de Wi-Fi**: Gerencia configurações e credenciais de rede sem fio---

- **Verificação de Fibra**: Teste de conectividade de rede

- **Configuração de Telefone**: Configuração VoIP e linha fixa com formulários melhorados

### 🎨 **UI/UX Moderno**

- **Gerenciamento VLAN/PPPoE**: Configuração de camada de rede</div>

- **Sistema de Tema Duplo**: Temas cyberpunk escuro e moderno claro

- **Ícones Geométricos**: Sistema de símbolos consistente ◯◪◈◐◊◇◎◆- **Controle de Acesso Web**: Senha de administrador e gerenciamento de acesso

- **Layout Responsivo**: Desktop lado a lado, empilhamento vertical mobile

- **Consistência de Botões**: Efeitos hover unificados, animações brilhantes e estilização- **Bridge ONT Nokia**: Verificação MAC integrada e geração de comandos## ✨ Overview

- **Rodapé Aprimorado**: Créditos do desenvolvedor com cores adaptáveis ao tema

- **Espaçamento Otimizado**: Componentes proporcionais e dimensionamento consistente### 📊 **Ferramentas Avançadas\*\***Nokia ONU Manager\*\* is a cutting-edge web application designed for managing Nokia Optical Network Unit (ONU) devices with stunning dual-theme aesthetics. Built with modern React technology, it provides network administrators with powerful tools to configure, monitor, and troubleshoot Nokia ONU equipment through an intuitive interface that adapts between cyberpunk dark mode and modern light mode.

## 🛠️ Stack Tecnológico- **Sistema de Conferência de Caixa**: Compara tabelas antes/depois com estilo de botão unificado## 🎨 Dual Theme System

- **Frontend**: React.js 18 com Vite 7.1.7 para desenvolvimento e build otimizados- **Detecção de Diferenças**: Destaque automático de mudanças em ambos os temas

- **Estilização**: CSS3 avançado com propriedades customizadas para temas duplos

- **Ícones**: Símbolos geométricos cyberpunk (◯◪◈◐◊◇◎◆) para branding consistente- **Rastreamento de Clientes Offline**: Monitora dispositivos offline com UI consistente### 🌙 **Dark Mode - Cyberpunk Aesthetic**

- **Temas**: Troca dinâmica entre tema escuro (ciano/magenta) e claro (verde lima #98c913)

- **Layout**: CSS Grid com desktop lado a lado responsivo e empilhamento mobile- **Exportação/Importação de Dados**: Backup e restauração de configurações

- **Gerenciamento de Estado**: Hooks do React com persistência localStorage para preferências de tema

- **Notificações**: Integração personalizada do SweetAlert2 com estilização adaptável ao tema- **Geração de Comandos**: Copia comandos de rede para a área de transferência com notificações- **Neon Colors**: Cyan (#00FFFF) and Magenta (#FF00FF) accents

- **Deploy**: GitHub Pages com pipeline automatizado do Actions

- **Desenvolvimento**: Hot Module Replacement (HMR) para feedback instantâneo no desenvolvimento- **Dark Backgrounds**: Deep blacks and dark blues

## 🏗️ Arquitetura### 🎯 **Operações Adicionais**- **Glow Effects**: Neon lighting and shadow effects

### Estrutura de Componentes- **Busca PON**: Exploração da topologia de rede- **Typography**: Futuristic Orbitron and Roboto Mono fonts

```

src/- **Monitoramento de Alarmes**: Rastreamento de alertas do sistema

├── components/          # Componentes de recursos

│   ├── PesquisaCliente.jsx      # Busca de clientes com formulários aprimorados- **Reinício da ONU**: Funcionalidade de reinicialização remota do dispositivo### ☀️ **Light Mode - Modern Green**

│   ├── PosicaoCliente.jsx       # Rastreamento de posição com atualizações em tempo real

│   ├── ProvisionarCliente.jsx   # Gerenciamento de provisionamento com notificações- **Solicitações de Provisionamento**: Manipula registros de novos dispositivos

│   ├── ConfiguracaoWifi.jsx     # Configuração Wi-Fi com validação

│   ├── ConfiguracaoTelefone.jsx # Configuração de telefone com consistência de formulário- **Verificação de Fibra**: Teste de conectividade de rede- **Primary Color**: Lime Green (#98c913)

│   ├── AlterarVlanPppoe.jsx     # Gerenciamento de camada de rede

│   ├── AlterarSenhaWeb.jsx      # Controle de acesso com segurança- **Clean Backgrounds**: Light grays and whites

│   ├── BridgeOntNokia.jsx       # Configuração bridge com verificação MAC

│   ├── ConferenciaCaixa.jsx     # Conferência de caixa com comparação de tabelas### 🎨 **UI/UX Moderno**- **Subtle Effects**: Green glow effects and shadows

│   └── OutrasOpcoes.jsx         # Utilitários adicionais e consulta

├── hooks/               # Hooks customizados do React- **Professional Look**: Clean, modern, energy-efficient design

│   ├── useSweetAlert.js         # Sistema de notificação com integração de tema

│   └── useTheme.js              # Gerenciamento e persistência de tema- **Sistema de Tema Duplo**: Temas cyberpunk escuro e moderno claro

├── services/            # Camadas de integração de API

│   ├── onuService.js            # Operações do dispositivo ONU- **Ícones Geométricos**: Sistema de símbolos consistente ◯◪◈◐◊◇◎◆### 🔄 **Theme Toggle**

│   ├── phoneService.js          # Chamadas de serviço de telefonia

│   └── wifiService.js           # APIs de configuração wireless- **Layout Responsivo**: Desktop lado a lado, empilhamento vertical mobile

└── utils/               # Funções utilitárias

    └── validation.js            # Validação de formulário e verificação de dados- **Consistência de Botões**: Efeitos hover unificados, animações brilhantes e estilização- **Floating Button**: Easy access theme switcher

```

- **Rodapé Aprimorado**: Créditos do desenvolvedor com cores adaptáveis ao tema- **Instant Switch**: Real-time theme transition

## 🚀 Primeiros Passos

- **Espaçamento Otimizado**: Componentes proporcionais e dimensionamento consistente- **Persistence**: Theme preference saved in localStorage

### Pré-requisitos

- **Universal**: Affects entire application including footer

- **Node.js** (v18 ou superior) - Necessário para recursos do React 18

- **npm** ou **yarn** - Gerenciador de pacotes## 🛠️ Stack Tecnológico

- **Navegador moderno** - Chrome, Firefox, Safari ou Edge

## 🚀 Features

### Instalação

- **Frontend**: React.js 18 com Vite 7.1.7 para desenvolvimento e build otimizados

1. **Clone o repositório**

- **Estilização**: CSS3 avançado com propriedades customizadas para temas duplos### 🔍 **Client Management**

  ````bash

  git clone https://github.com/MatheusAraujo86/obscure-xylophone.git- **Ícones**: Símbolos geométricos cyberpunk (◯◪◈◐◊◇◎◆) para branding consistente

  cd onu-nokia-react

  ```- **Temas**: Troca dinâmica entre tema escuro (ciano/magenta) e claro (verde lima #98c913)- **Advanced Client Search**: Locate clients by various parameters with geometric cyberpunk icons
  ````

2. **Instale as dependências**- **Layout**: CSS Grid com desktop lado a lado responsivo e empilhamento mobile- **Client Positioning**: Real-time slot/GPON/index tracking in side-by-side layout

   ````bash- **Gerenciamento de Estado**: Hooks do React com persistência localStorage para preferências de tema- **Comprehensive Client Data**: View detailed client information with theme-adaptive colors

   npm install

   ```- **Notificações**: Integração personalizada do SweetAlert2 com estilização adaptável ao tema

   ````

3. **Inicie o servidor de desenvolvimento**- **Deploy**: GitHub Pages com pipeline automatizado do Actions### ⚙️ **Device Configuration**

   ````bash- **Desenvolvimento**: Hot Module Replacement (HMR) para feedback instantâneo no desenvolvimento

   npm run dev

   ```- **Client Provisioning/Deprovisioning**: Complete lifecycle management with enhanced UI

   ````

4. **Abra seu navegador**## 🎨 Sistema de Temas- **Wi-Fi Configuration**: Manage wireless settings and credentials

   ```

   http://localhost:5173- **Phone Configuration**: VoIP and landline setup with improved forms

   ```

Nosso sistema de tema duplo oferece duas experiências distintas:- **VLAN/PPPoE Management**: Network layer configuration

A aplicação abrirá automaticamente com hot reloading habilitado.

- **Web Access Control**: Admin password and access management

### Build para Produção

### 🌙 **Modo Escuro - Cyberpunk**- **Bridge ONT Nokia**: Integrated MAC verification and command generation

````bash

# Build da aplicação- **Cores Primárias**: Ciano elétrico (#00FFFF) e magenta vibrante (#FF00FF)

npm run build

- **Fundo**: Pretos espaciais profundos com gradientes sutis### 📊 **Advanced Tools**

# Preview do build de produção localmente

npm run preview- **Tipografia**: Orbitron para cabeçalhos, Roboto Mono para elementos de código



# Deploy para GitHub Pages (se configurado)- **Efeitos**: Brilho neon, bordas elétricas e elementos holográficos- **Box Conference System**: Compare before/after tables with unified button styling

npm run deploy

```- **Estética**: Cyberpunk futurista com linguagem visual high-tech- **Difference Detection**: Automatic highlighting of changes in both themes



## 🎯 Guia de Uso- **Down Clients Tracking**: Monitor offline devices with consistent UI



### 🌓 **Troca de Tema**### ☀️ **Modo Claro - Verde Moderno**- **Data Export/Import**: Backup and restore configurations

- Clique no botão alternador de tema flutuante (🌙/☀️) no canto superior direito

- Modo escuro: Estética cyberpunk com cores ciano/magenta- **Cores Primárias**: Verde lima fresco (#98c913) com variações naturais- **Command Generation**: Copy network commands to clipboard with notifications

- Modo claro: Visual profissional moderno com acentos verde lima

- Preferência de tema é salva automaticamente- **Fundo**: Brancos limpos e cinzas suaves para visual profissional



### 🔍 **Operações de Cliente**- **Tipografia**: Mesmo sistema de fontes geométricas adaptado para fundos claros### � **Additional Operations**

1. **Buscar Clientes**: Use o formulário de busca com vários parâmetros

2. **Ver Posição**: Verifique informações de slot/GPON/índice do cliente em tempo real- **Efeitos**: Sombras sutis, bordas limpas e gradientes modernos

3. **Provisionar/Desprovisionar**: Gerencie o ciclo de vida do cliente com formulários aprimorados

4. **Configurar Serviços**: Configure Wi-Fi, telefone e configurações de rede- **Estética**: Corporativo contemporâneo com acentos verdes eco-friendly- **PON Search**: Network topology exploration



### 📊 **Ferramentas Avançadas**- **Alarm Monitoring**: System alert tracking

- **Conferência de Caixa**: Compare configurações antes/depois com visualização de tabela

- **Bridge ONT**: Configure dispositivos bridge Nokia com verificação MACAmbos os temas apresentam:- **ONU Restart**: Remote device reboot functionality

- **Opções Adicionais**: Acesse ferramentas de consulta e utilitários de rede

- **Armazenamento Persistente**: Sua escolha de tema salva automaticamente- **Provisioning Requests**: Handle new device registrations

### 💡 **Dicas**

- Todos os formulários incluem validação em tempo real- **Alternador Flutuante**: Troca fácil de tema com transição animada- **Fiber Verification**: Network connectivity testing

- Use os ícones geométricos como auxílios de navegação visual

- Copie comandos diretamente para a área de transferência com um clique- **Consistência de Componentes**: Todos os elementos se adaptam perfeitamente

- Todas as mudanças são confirmadas com notificações temáticas

- **Padronização de Botões**: Efeitos hover uniformes, animações brilhantes e estilização em todos os componentes### 🎨 **Modern UI/UX**

## 🌐 Deploy

## 🏗️ Arquitetura- **Dual Theme System**: Dark cyberpunk and light modern themes

### **🚀 GitHub Pages (Automatizado)**

- **Geometric Icons**: Consistent ◯◪◈◐◊◇◎◆ symbol system

Este projeto inclui deploy automático para GitHub Pages:

### Estrutura de Componentes- **Responsive Layout**: Desktop side-by-side, mobile vertical stacking

1. **Configuração do Repositório**:

   - Fonte das Pages: GitHub Actions```- **Button Consistency**: Unified hover effects, shine animations, and styling

   - URL Base: `/obscure-xylophone/` (configurado em `vite.config.js`)

src/- **Enhanced Footer**: Developer credits with theme-adaptive colors

2. **Deploy Automático**:

   ```bash├── components/          # Componentes de recursos- **Optimized Spacing**: Proportional components and consistent sizing

   git add .

   git commit -m "Deploy para GitHub Pages" │   ├── PesquisaCliente.jsx      # Busca de clientes com formulários aprimorados- **Responsive Design**: Mobile and desktop optimized

   git push origin master

   ```│   ├── PosicaoCliente.jsx       # Rastreamento de posição com atualizações em tempo real



3. **Site ao Vivo**: https://MatheusAraujo86.github.io/obscure-xylophone/│   ├── ProvisionarCliente.jsx   # Gerenciamento de provisionamento com notificações## 🛠️ Tech Stack



### **🔧 Build e Deploy Manual**│   ├── ConfiguracaoWifi.jsx     # Configuração Wi-Fi com validação



```bash│   ├── ConfiguracaoTelefone.jsx # Configuração de telefone com consistência de formulário- **Frontend**: React.js 18 with Vite 7.1.7 for optimized development and build

# Build para produção

npm run build│   ├── AlterarVlanPppoe.jsx     # Gerenciamento de camada de rede- **Styling**: Advanced CSS3 with custom properties for dual theming



# Preview local│   ├── AlterarSenhaWeb.jsx      # Controle de acesso com segurança- **Icons**: Geometric cyberpunk symbols (◯◪◈◐◊◇◎◆) for consistent branding

npm run preview

│   ├── BridgeOntNokia.jsx       # Configuração bridge com verificação MAC- **Themes**: Dynamic dark (cyan/magenta) and light (green lime #98c913) theme switching

# Deploy manual (se necessário)

npm run deploy│   ├── ConferenciaCaixa.jsx     # Conferência de caixa com comparação de tabelas- **Layout**: CSS Grid with responsive side-by-side desktop and mobile stacking

````

│ └── OutrasOpcoes.jsx # Utilitários adicionais e consulta- **State Management**: React hooks with localStorage persistence for theme preferences

## 📝 Diretrizes de Desenvolvimento

├── hooks/ # Hooks customizados do React- **Notifications**: Custom SweetAlert2 integration with theme-adaptive styling

### **Estilo de Código**

- **Componentes**: Componentes funcionais com hooks│ ├── useSweetAlert.js # Sistema de notificação com integração de tema- **Deployment**: GitHub Pages with automated Actions pipeline

- **Estilização**: Propriedades customizadas CSS para temas

- **Ícones**: Símbolos geométricos (◯◪◈◐◊◇◎◆) para consistência│ └── useTheme.js # Gerenciamento e persistência de tema- **Development**: Hot Module Replacement (HMR) for instant development feedback

- **Layout**: CSS Grid para design responsivo

- **Estado**: Estado local com hooks, localStorage para persistência├── services/ # Camadas de integração de API

### **Desenvolvimento de Temas**│ ├── onuService.js # Operações do dispositivo ONU## 🎨 Theme System

- Use propriedades customizadas CSS: `--accent-primary`, `--accent-secondary`, `--text-button`

- Teste ambos os temas claro e escuro para qualquer novo componente│ ├── phoneService.js # Chamadas de serviço de telefonia

- Garanta que a estilização de botões siga o sistema padronizado

- Verifique comportamento responsivo em desktop lado a lado e empilhamento mobile│ └── wifiService.js # APIs de configuração wirelessOur dual theme system provides two distinct experiences:

### **Adicionando Novos Componentes**└── utils/ # Funções utilitárias

1. Crie componente em `src/components/`

2. Importe e adicione ao layout do `App.jsx` └── validation.js # Validação de formulário e verificação de dados### 🌙 **Dark Mode - Cyberpunk**

3. Aplique variáveis de tema do `App.css`

4. Use ícones geométricos para consistência```

5. Teste em ambos os modos de tema

6. Garanta comportamento responsivo- **Primary Colors**: Electric cyan (#00FFFF) and vibrant magenta (#FF00FF)

## 🤝 Contribuindo## 🚀 Primeiros Passos- **Background**: Deep space blacks with subtle gradients

### **Fluxo de Trabalho de Desenvolvimento**- **Typography**: Orbitron for headers, Roboto Mono for code elements

1. Faça fork do repositório

2. Crie uma branch de funcionalidade: `git checkout -b nome-da-funcionalidade`### Pré-requisitos- **Effects**: Neon glow, electric borders, and holographic elements

3. Faça mudanças seguindo as diretrizes de estilo de código

4. Teste em ambos os temas claro e escuro- **Aesthetic**: Futuristic cyberpunk with high-tech visual language

5. Garanta que o design responsivo funciona

6. Commit das mudanças: `git commit -m "Adicionar descrição da funcionalidade"`- **Node.js** (v18 ou superior) - Necessário para recursos do React 18

7. Push para branch: `git push origin nome-da-funcionalidade`

8. Crie um Pull Request- **npm** ou **yarn** - Gerenciador de pacotes### ☀️ **Light Mode - Modern Green**

### **Relatórios de Bug**- **Navegador moderno** - Chrome, Firefox, Safari ou Edge

- Use GitHub Issues para relatórios de bug

- Inclua versão do navegador e modo de tema- **Primary Colors**: Fresh lime green (#98c913) with natural variations

- Forneça passos para reproduzir o problema

- Inclua screenshots se aplicável### Instalação- **Background**: Clean whites and soft grays for professional look

## 📄 Licença- **Typography**: Same geometric font system adapted for light backgrounds

Licença MIT - Sinta-se livre para usar este projeto para fins de aprendizado e desenvolvimento.1. **Clone o repositório**- **Effects**: Subtle shadows, clean borders, and modern gradients

## 👥 Autores- **Aesthetic**: Contemporary corporate with eco-friendly green accents

- **Matheus** - Desenvolvedor Principal & Designer UI/UX ````bash

- **Esteban** - Integração Backend & Arquitetura do Sistema

  git clone https://github.com/MatheusAraujo86/obscure-xylophone.gitBoth themes feature:

## 🙏 Agradecimentos

cd onu-nokia-react

- Comunidade React.js pela documentação excelente

- Equipe Vite pela experiência de desenvolvimento ultra-rápida ```- **Persistent Storage**: Your theme choice saves automatically

- SweetAlert2 pelas notificações bonitas

- GitHub Pages pela hospedagem gratuita ````

- Inspiração estética cyberpunk das tendências de design futurista

- Princípios de web design moderno para o sistema de tema duplo- **Floating Toggle**: Easy theme switching with animated transition

## 📞 Suporte2. **Instale as dependências**- **Component Consistency**: All elements adapt seamlessly

Para perguntas, solicitações de recursos ou suporte:- **Button Standardization**: Unified hover effects, shine animations, and styling across all components

- 🐛 **Issues**: [GitHub Issues](https://github.com/MatheusAraujo86/obscure-xylophone/issues)

- 📧 **Email**: Entre em contato com os proprietários do repositório ```bash

- 💬 **Discussões**: GitHub Discussions (se habilitado)

  npm install## 🏗️ Architecture

---

`````

<div align="center">

### Component Structure

**🌟 Dê uma estrela neste repositório se ele te ajudou!**

3. **Inicie o servidor de desenvolvimento**

![GitHub stars](https://img.shields.io/github/stars/MatheusAraujo86/obscure-xylophone?style=social)

![GitHub forks](https://img.shields.io/github/forks/MatheusAraujo86/obscure-xylophone?style=social)````



*Construído com ❤️ usando React e tecnologias web modernas*   ```bashsrc/



</div>   npm run dev├── components/          # Feature components

 ```│   ├── PesquisaCliente.jsx      # Client search with enhanced forms

│   ├── PosicaoCliente.jsx       # Position tracking with real-time updates

4. **Abra seu navegador**│   ├── ProvisionarCliente.jsx   # Provisioning management with notifications

 ```│   ├── ConfiguracaoWifi.jsx     # Wi-Fi configuration with validation

 http://localhost:5173│   ├── ConfiguracaoTelefone.jsx # Phone setup with form consistency

 ```│   ├── AlterarVlanPppoe.jsx     # Network layer management

│   ├── AlterarSenhaWeb.jsx      # Access control with security

 A aplicação abrirá automaticamente com hot reloading habilitado.│   ├── BridgeOntNokia.jsx       # Bridge configuration with MAC verification

│   ├── ConferenciaCaixa.jsx     # Box conference with table comparison

### Build para Produção│   └── OutrasOpcoes.jsx         # Additional utilities and consultation

├── hooks/               # Custom React hooks

```bash│   ├── useSweetAlert.js         # Notification system with theme integration

# Build da aplicação│   └── useTheme.js             # Theme management and persistence

npm run build├── services/            # API integration layers

│   ├── onuService.js           # ONU device operations

# Preview do build de produção localmente│   ├── phoneService.js         # Telephony service calls

npm run preview│   └── wifiService.js          # Wireless configuration APIs

└── utils/               # Utility functions

# Deploy para GitHub Pages (se configurado)    └── validation.js           # Form validation and data checking

npm run deploy```

`````

## 🚀 Getting Started

## 🎯 Guia de Uso

### Prerequisites

### 🌓 **Troca de Tema**

- Clique no botão alternador de tema flutuante (🌙/☀️) no canto superior direito- **Node.js** (v18 or higher) - Required for React 18 features

- Modo escuro: Estética cyberpunk com cores ciano/magenta- **npm** or **yarn** - Package manager

- Modo claro: Visual profissional moderno com acentos verde lima- **Modern web browser** - Chrome, Firefox, Safari, or Edge

- Preferência de tema é salva automaticamente

### Installation

### 🔍 **Operações de Cliente**

1. **Buscar Clientes**: Use o formulário de busca com vários parâmetros1. **Clone the repository**

2. **Ver Posição**: Verifique informações de slot/GPON/índice do cliente em tempo real

3. **Provisionar/Desprovisionar**: Gerencie o ciclo de vida do cliente com formulários aprimorados ```bash

4. **Configurar Serviços**: Configure Wi-Fi, telefone e configurações de rede git clone https://github.com/MatheusAraujo86/obscure-xylophone.git

   cd onu-nokia-react

### 📊 **Ferramentas Avançadas** ```

- **Conferência de Caixa**: Compare configurações antes/depois com visualização de tabela

- **Bridge ONT**: Configure dispositivos bridge Nokia com verificação MAC2. **Install dependencies**

- **Opções Adicionais**: Acesse ferramentas de consulta e utilitários de rede

  ```bash

  ```

### 💡 **Dicas** npm install

- Todos os formulários incluem validação em tempo real ```

- Use os ícones geométricos como auxílios de navegação visual

- Copie comandos diretamente para a área de transferência com um clique3. **Start development server**

- Todas as mudanças são confirmadas com notificações temáticas

  ```bash

  ```

## 🌐 Deploy npm run dev

```

### **🚀 GitHub Pages (Automatizado)**

4. **Open your browser**

Este projeto inclui deploy automático para GitHub Pages:

```

1. **Configuração do Repositório**: http://localhost:5173

   - Fonte das Pages: GitHub Actions ```

   - URL Base: `/obscure-xylophone/` (configurado em `vite.config.js`)

   The application will automatically open with hot reloading enabled.

2. **Deploy Automático**:

   ````bash### Build for Production

   git add .

   git commit -m "Deploy para GitHub Pages" ```bash

   git push origin master# Build the application

   ```npm run build

   ````

3. **Site ao Vivo**: https://MatheusAraujo86.github.io/obscure-xylophone/# Preview production build locally

npm run preview

### **🔧 Build e Deploy Manual**

# Deploy to GitHub Pages (if configured)

````bashnpm run deploy

# Build para produção```

npm run build

## 🎯 Usage Guide

# Preview local

npm run preview  ### 🌓 **Theme Switching**



# Deploy manual (se necessário)- Click the floating theme toggle button (🌙/☀️) in the top-right corner

npm run deploy- Dark mode: Cyberpunk aesthetic with cyan/magenta colors

```- Light mode: Modern professional look with green lime accents

- Theme preference is automatically saved

## 📝 Diretrizes de Desenvolvimento

### 🔍 **Client Operations**

### **Estilo de Código**

- **Componentes**: Componentes funcionais com hooks1. **Search Clients**: Use the search form with various parameters

- **Estilização**: Propriedades customizadas CSS para temas2. **View Position**: Check real-time client slot/GPON/index information

- **Ícones**: Símbolos geométricos (◯◪◈◐◊◇◎◆) para consistência3. **Provision/Deprovision**: Manage client lifecycle with enhanced forms

- **Layout**: CSS Grid para design responsivo4. **Configure Services**: Set up Wi-Fi, phone, and network settings

- **Estado**: Estado local com hooks, localStorage para persistência

### 📊 **Advanced Tools**

### **Desenvolvimento de Temas**

- Use propriedades customizadas CSS: `--accent-primary`, `--accent-secondary`, `--text-button`- **Box Conference**: Compare before/after configurations with table view

- Teste ambos os temas claro e escuro para qualquer novo componente- **Bridge ONT**: Configure Nokia bridge devices with MAC verification

- Garanta que a estilização de botões siga o sistema padronizado- **Additional Options**: Access consultation tools and network utilities

- Verifique comportamento responsivo em desktop lado a lado e empilhamento mobile

### 💡 **Tips**

### **Adicionando Novos Componentes**

1. Crie componente em `src/components/`- All forms include real-time validation

2. Importe e adicione ao layout do `App.jsx`- Use the geometric icons as visual navigation aids

3. Aplique variáveis de tema do `App.css`- Copy commands directly to clipboard with one click

4. Use ícones geométricos para consistência- All changes are confirmed with themed notifications

5. Teste em ambos os modos de tema

6. Garanta comportamento responsivo## 🌐 Deployment



## 🤝 Contribuindo### **🚀 GitHub Pages (Automated)**



### **Fluxo de Trabalho de Desenvolvimento**This project includes automatic deployment to GitHub Pages:

1. Faça fork do repositório

2. Crie uma branch de funcionalidade: `git checkout -b nome-da-funcionalidade`1. **Repository Setup**:

3. Faça mudanças seguindo as diretrizes de estilo de código

4. Teste em ambos os temas claro e escuro   - Pages source: GitHub Actions

5. Garanta que o design responsivo funciona   - Base URL: `/obscure-xylophone/` (configured in `vite.config.js`)

6. Commit das mudanças: `git commit -m "Adicionar descrição da funcionalidade"`

7. Push para branch: `git push origin nome-da-funcionalidade` 2. **Automatic Deployment**:

8. Crie um Pull Request

   ```bash

### **Relatórios de Bug**   git add .

- Use GitHub Issues para relatórios de bug   git commit -m "Deploy to GitHub Pages"

- Inclua versão do navegador e modo de tema   git push origin master

- Forneça passos para reproduzir o problema   ```

- Inclua screenshots se aplicável

3. **Live Site**: https://MatheusAraujo86.github.io/obscure-xylophone/

## 📄 Licença

### **🔧 Manual Build & Deploy**

Licença MIT - Sinta-se livre para usar este projeto para fins de aprendizado e desenvolvimento.

```bash

## 👥 Autores# Build for production

npm run build

- **Matheus** - Desenvolvedor Principal & Designer UI/UX

- **Esteban** - Integração Backend & Arquitetura do Sistema# Preview locally

npm run preview

## 🙏 Agradecimentos

# Manual deploy (if needed)

- Comunidade React.js pela documentação excelentenpm run deploy

- Equipe Vite pela experiência de desenvolvimento ultra-rápida```

- SweetAlert2 pelas notificações bonitas

- GitHub Pages pela hospedagem gratuita## 📝 Development Guidelines

- Inspiração estética cyberpunk das tendências de design futurista

- Princípios de web design moderno para o sistema de tema duplo### **Code Style**



## 📞 Suporte- **Components**: Functional components with hooks

- **Styling**: CSS custom properties for theming

Para perguntas, solicitações de recursos ou suporte:- **Icons**: Geometric symbols (◯◪◈◐◊◇◎◆) for consistency

- 🐛 **Issues**: [GitHub Issues](https://github.com/MatheusAraujo86/obscure-xylophone/issues)- **Layout**: CSS Grid for responsive design

- 📧 **Email**: Entre em contato com os proprietários do repositório- **State**: Local state with hooks, localStorage for persistence

- 💬 **Discussões**: GitHub Discussions (se habilitado)

### **Theme Development**

---

- Use CSS custom properties: `--accent-primary`, `--accent-secondary`, `--text-button`

<div align="center">- Test both light and dark themes for any new components

- Ensure button styling follows the standardized system

**🌟 Dê uma estrela neste repositório se ele te ajudou!**- Verify responsive behavior on desktop side-by-side and mobile stacking



![GitHub stars](https://img.shields.io/github/stars/MatheusAraujo86/obscure-xylophone?style=social)### **Adding New Components**

![GitHub forks](https://img.shields.io/github/forks/MatheusAraujo86/obscure-xylophone?style=social)

1. Create component in `src/components/`

*Construído com ❤️ usando React e tecnologias web modernas*2. Import and add to `App.jsx` layout

3. Apply theme variables from `App.css`

</div>4. Use geometric icons for consistency
5. Test in both theme modes
6. Ensure responsive behavior

## 🤝 Contributing

### **Development Workflow**

1. Fork the repository
2. Create a feature branch: `git checkout -b feature-name`
3. Make changes following the code style guidelines
4. Test in both light and dark themes
5. Ensure responsive design works
6. Commit changes: `git commit -m "Add feature description"`
7. Push to branch: `git push origin feature-name`
8. Create a Pull Request

### **Bug Reports**

- Use GitHub Issues for bug reports
- Include browser version and theme mode
- Provide steps to reproduce the issue
- Include screenshots if applicable

## � License

MIT License - Feel free to use this project for learning and development purposes.

## 👥 Authors

- **Matheus** - Lead Developer & UI/UX Designer
- **Esteban** - Backend Integration & System Architecture

## 🙏 Acknowledgments

- React.js community for excellent documentation
- Vite team for lightning-fast development experience
- SweetAlert2 for beautiful notifications
- GitHub Pages for free hosting
- Cyberpunk aesthetic inspiration from futuristic design trends
- Modern web design principles for the dual theme system

## 📞 Support

For questions, feature requests, or support:

- 🐛 **Issues**: [GitHub Issues](https://github.com/MatheusAraujo86/obscure-xylophone/issues)
- 📧 **Email**: Contact repository owners
- � **Discussions**: GitHub Discussions (if enabled)

---

<div align="center">

**🌟 Star this repository if it helped you!**

![GitHub stars](https://img.shields.io/github/stars/MatheusAraujo86/obscure-xylophone?style=social)
![GitHub forks](https://img.shields.io/github/forks/MatheusAraujo86/obscure-xylophone?style=social)

_Built with ❤️ using React and modern web technologies_

</div>
- Export comparison results

### 🎨 **UI Navigation**

- **Sidebar Menu**: Access all tools and configurations
- **Fixed Positioning**: Client position remains visible across tools
- **Full-Width Mode**: Conference tool automatically expands for better visibility
- **Responsive Design**: Works on mobile and desktop devices

## 🧪 Testing

```bash
# Run linting
npm run lint

# Build and test locally
npm run build && npm run preview
````

## 🤝 Contributing

We welcome contributions! Please follow these steps:

1. **Fork the repository**
2. **Create a feature branch**: `git checkout -b feature/amazing-feature`
3. **Make your changes** and follow the cyberpunk design system
4. **Test thoroughly**: Ensure all features work correctly
5. **Commit your changes**: `git commit -m 'Add amazing feature'`
6. **Push to the branch**: `git push origin feature/amazing-feature`
7. **Open a Pull Request**

### 🎨 Design Guidelines

- Maintain the cyberpunk aesthetic (cyan/magenta color scheme)
- Use Orbitron/Roboto Mono fonts for consistency
- Include glow effects and smooth animations
- Ensure responsive design principles

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 👤 Author

**Higor Brito & Matheus Araujo**

- GitHub: [@MatheusAraujo86](https://github.com/MatheusAraujo86)
- Company: Solução Network

## 🙏 Acknowledgments

- Nokia for ONU technology and documentation
- React team for the amazing framework
- Vite for lightning-fast development experience
- GitHub Pages for free hosting
- Cyberpunk aesthetic inspiration from futuristic design trends

## 🆘 Support

If you encounter any issues or have questions:

1. **Check the Issues tab** for existing solutions
2. **Create a new issue** with detailed information
3. **Include system information** and error messages
4. **Describe the expected vs actual behavior**

---

<div align="center">

**Made with ❤️ and lots of ⚡ neon lights**

_Transform your Nokia ONU management experience with cyberpunk style!_ 🔮

**🌍 [View Live Demo](https://MatheusAraujo86.github.io/obscure-xylophone/)**

</div>
# Deploy com correções de zoom implementadas
