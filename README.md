# Ligeirinho - Cena Gráfica 2D Interativa

## Alunos:
 - Adrian Modesto Lauzid
 - Artur Vinícius Lima Ramos da Silva
 - Gustavo dos Santos Silva
 - Lucas Pereira de Souza

## Descrição do Projeto

Este projeto é uma aplicação interativa de computação gráfica 2D desenvolvida em Vue.js que apresenta o Ligeirinho em uma cena animada que demonstra conceitos fundamentais de computação gráfica, incluindo rasterização, projeção, iluminação local e transformações afins.

## Objetivos Acadêmicos

O projeto foi desenvolvido como parte da disciplina de **Computação Gráfica** e visa demonstrar na prática os seguintes conceitos:

- **Espaço Virtual Estendido**: Mundo virtual de dimensões maiores que a janela de visualização
- **Sistema de Câmera**: Projeção de uma subárea do espaço virtual para a tela
- **Iluminação Phong**: Modelo de iluminação local com componentes ambiente, difusa e especular
- **Transformações Afins**: Translação, rotação e escala aplicadas ao personagem
- **Animação Autônoma**: Movimento inteligente com desvio de obstáculos
- **Interatividade**: Controles de câmera e interação com o personagem

## Funcionalidades Implementadas

### 1. Sistema de Espaço Virtual
- **Dimensões**: 1600×1200 pixels (2× resolução da câmera)
- **Câmera Fixa**: 800×600 pixels de resolução
- **Projeção**: Ortográfica com movimentação suave da câmera
- **Independência**: Câmera e personagem operam de forma completamente independente

### 2. Personagem Ligeirinho
- **Design Visual**: Rato amarelo distintivo com sombrero, cauda animada e pernas correndo
- **"IA" Autônoma**: Movimento randomizado com mudanças de direção a cada poucos segundos
- **Desvio de Obstáculos**: Navegação inteligente ao redor de paredes e limites
- **Transformações**: Rotação, translação e escala para animação fluida

### 3. Modelo de Iluminação Phong
- **Implementação Completa**: Componentes ambiente, difusa e especular
- **Luz Direcional**: Fonte de luz configurável afetando todos os objetos
- **Cores Dinâmicas**: Cálculos de cor em tempo real para personagem e ambiente
- **Profundidade Visual**: Efeitos de iluminação 3D simulados em objetos 2D

### 4. Sistema de Animação
- **Movimento Suave**: Aceleração/desaceleração com movimento baseado em física
- **Animação por Frames**: Sprite do personagem com ondulação da cauda e movimento das pernas
- **Efeitos de Velocidade**: Linhas de velocidade dinâmicas quando correndo rápido
- **Variações de Trajetória**: Mudanças contínuas de caminho com desvio de obstáculos

### 5. Ambiente
- **Fundos com Gradiente**: Transições multicoloridas de céu para chão
- **Grid de Referência**: Orientação espacial com padrão de grade animado
- **Obstáculos Estáticos**: Paredes e plataformas com iluminação adequada
- **Minimapa**: Visão em tempo real de todo o espaço virtual

### 6. Recursos Interativos
- **Controles de Câmera**: WASD ou setas do teclado para movimento suave da câmera
- **Interação com Personagem**: Mecânica de clique-para-capturar que faz o Ligeirinho pular
- **Interface em Tempo Real**: Rastreamento de posição, descrições de recursos e feedback visual
- **Design Responsivo**: Escala adequada e efeitos visuais

## Tecnologias Utilizadas

- **Vue.js 3**: Framework frontend reativo
- **Vite**: Ferramenta de build rápida e moderna
- **Canvas API**: Renderização gráfica 2D nativa
- **JavaScript ES6+**: Programação orientada a objetos e funcional
- **CSS3**: Estilização avançada com gradientes e efeitos

## Como Usar

### Instalação
```bash
# Clone o repositório
git clone https://github.com/ArtuurVinicius/CG-N2-PT2.git

# Navegue até o diretório
cd CG-N2-PT2

# Instale as dependências
npm install

# Execute o servidor de desenvolvimento
npm run dev
```

### Controles
1. **Movimento da Câmera**: Use WASD ou as setas do teclado para mover a câmera pelo espaço virtual
2. **Interação com Personagem**: Clique no Ligeirinho para fazê-lo pular para longe
3. **Observar Animação**: Veja o movimento autônomo, desvio de obstáculos e efeitos de iluminação
4. **Monitorar Progresso**: Use o minimapa para ver todo o mundo virtual e posições atuais

## Conceitos de Computação Gráfica Demonstrados

### Rasterização
- Renderização de sprites e vetores 2D
- Conversão de primitivas geométricas para pixels

### Projeção
- Transformação de coordenadas virtual-para-tela
- Sistema de câmera com viewport fixo

### Iluminação Local (Phong)
- **Componente Ambiente**: Iluminação uniforme base
- **Componente Difusa**: Reflexão dependente do ângulo da superfície
- **Componente Especular**: Reflexões brilhantes direcionais
- Cálculo de intensidade final combinando todos os componentes

### Transformações Afins
- **Translação**: Movimento de posição
- **Rotação**: Mudanças de orientação
- **Escala**: Alterações de tamanho
- Aplicadas em tempo real durante a animação

### Espaço Virtual Estendido
- Mundo 2× maior que a resolução da câmera
- Navegação contínua através do espaço

### Sistemas Independentes
- Câmera e personagem operam separadamente
- Sincronização adequada entre sistemas

## Estrutura do Projeto

```
CG-N2-PT2/
├── src/
│   ├── App.vue          # Componente principal com toda a lógica gráfica
│   ├── main.js          # Ponto de entrada da aplicação
│   └── style.css        # Estilos globais
├── public/              # Arquivos estáticos
├── package.json         # Dependências do projeto
├── vite.config.js       # Configuração do Vite
└── README.md           # Documentação do projeto
```

## Contexto Acadêmico

Este projeto foi desenvolvido como atividade avaliativa da disciplina de **Computação Gráfica** e demonstra:

- Compreensão prática de algoritmos de renderização 2D
- Implementação de modelos de iluminação realistas
- Aplicação de transformações matemáticas para animação
- Desenvolvimento de sistemas interativos em tempo real
- Integração de múltiplos conceitos de computação gráfica em uma aplicação coesa
