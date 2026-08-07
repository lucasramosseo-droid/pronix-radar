# PRONIX Radar

Radar de produtos mais buscados no Mercado Livre para análise de demanda e oportunidade. Painel de uso interno da equipe Pronix.

## Novidade: dados reais do Mercado Livre

Desde esta atualização, o painel passa a exibir **dados reais do Mercado Livre** (preço, avaliações, tendências) sem exigir login dos especialistas. Antes, os valores eram estimativas locais; agora são coletados da API oficial do ML por um servidor intermediário.

### Como funciona

- Uma **Netlify Function** (`ml-bridge`) na URL `https://pronix-radar.netlify.app/.netlify/functions/ml-bridge` faz o papel de proxy seguro para a API do Mercado Livre.
- O **token de acesso** (autorização única feita pelo administrador) é guardado no **Netlify Blobs** e **renova automaticamente** a cada ~6h graças ao escopo `offline_access` — sem intervenção manual.
- O painel consulta o proxy com uma chave de API própria; os especialistas **não fazem login** no Mercado Livre.

### Infraestrutura

| Item | Valor |
| --- | --- |
| Painel (Netlify) | `https://pronix-radar.netlify.app/` |
| Redirect do GitHub Pages | `https://lucasramosseo-droid.github.io/pronix-radar/` (redireciona para o painel) |
| Código-fonte | `https://github.com/lucasramosseo-droid/pronix-radar-source` (privado) |
| Proxy (Netlify Function) | `https://pronix-radar.netlify.app/.netlify/functions/ml-bridge` |
| Aplicação Mercado Livre | PRONIX RADAR (client_id `3926651658690238`), com escopos `read`, `write` e `offline_access` |
| Armazenamento do token | Netlify Blobs, store `site:mlp-radar`, chave `token` |

### Frequência de atualização dos dados

Os dados reais são atualizados automaticamente, sem intervenção manual:

| Tipo de dado | Frequência |
| --- | --- |
| Tendências de busca | a cada 1 hora |
| Produtos / detalhes (preço, estoque) | a cada 6 horas |
| Avaliações | a cada 24 horas |

- O painel, enquanto aberto, re-consulta o proxy a cada 15 minutos; os valores vêm do cache do servidor.
- O cache expira sozinho e o proxy busca dados novos do Mercado Livre, dentro do limite da aplicação (18.000 requisições/hora).

### Segurança

- O `client_secret` do app ML e a chave do painel vivem apenas nas variáveis de ambiente da Netlify Function, nunca no HTML público.
- O proxy valida a chave de API (`ML_API_KEY`) em toda chamada de dados.
- As respostas do ML são armazenadas em cache no Blobs (conforme a tabela de frequência acima) para reduzir chamadas.

### Ações do proxy

- `status` — configuração e validade do token.
- `search` — busca de produtos por termo (`q`).
- `items` — itens de um produto/catálogo (`id`).
- `reviews` — avaliações de um item (`id`).
- `trends` — tendências de busca do site MLB.

## Uso

Abra `https://pronix-radar.netlify.app/` em qualquer navegador. Não requer instalação. Funciona online (para dados reais via proxy) e mantém estimativas locais como fallback offline.

## Experiência do painel

- **Status de conexão**: chip no cabeçalho mostra se o proxy está conectado ao Mercado Livre (verde = conectado, vermelho = offline).
- **Selo de atualização**: no Painel, um selo informa há quanto tempo os dados reais foram buscados no ML pela última vez.
- **Layout mobile**: em telas até 768px, o painel ganha um layout dedicado com navegação fixa inferior (bottom nav) e cards em coluna, otimizado para uso no celular.
- **Carregamento**: skeleton animado é exibido enquanto os dados reais são buscados.

## Atualizações recentes

### 07/08/2026 — Swipe de categorias fluido como Instagram (VERS 15)

- **Sem recarga ao arrastar para o lado no celular**: o gesto nativo de "voltar/avançar" do navegador (edge swipe de iOS/Android) recarregava a página ao deslizar o dedo na horizontal. Agora o painel detecta a intenção do gesto e bloqueia esse reload, mantendo o scroll normal.
- **Transição de slide real**: ao trocar de categoria por swipe, a tela atual sai na direção do dedo e a nova desliza de lado (não é mais um simples fade) — navegação fluida entre Painel, Departamentos, Produtos e Oportunidades.

### 07/08/2026 — Avaliações agregadas e cenários de vendas (VERS 15)

- **Avaliações somadas de vários vendedores**: a estimativa de vendas agora soma as avaliações de **até 3 anúncios** do catálogo (antes usava só o anúncio nº 1, o que subestimava produtos fortes). A nota média passa a ser a **média ponderada** pela quantidade de avaliações.
- **Cenários conservador / esperado / otimista**: cada produto com avaliações reais mostra a faixa de vendas/mês e faturamento/mês (taxa de avaliação de 25% / 15% / 8%), em vez de um número pontual — mais honesto para decidir.

### 07/08/2026 — Métricas reais por avaliações (VERS 14)

- **Vendas estimadas a partir de avaliações reais**: `vendas/dia = avaliações ÷ dias desde a 1ª avaliação ÷ taxa de avaliação (15%)`. Antes eram simuladas; agora têm origem em dados reais da API do ML.
- **Idade do produto**: a bridge devolve a data da **primeira avaliação** de cada item, e o painel mostra "1ª avaliação" e "Idade (dias)" no bloco de análise (no lugar da antiga linha genérica de catálogo).
- **Constantes declaradas**: conversão (5%), comissão ML (16%) e imposto (14,42%) são premissas explícitas, sem valores aleatórios. Sem avaliações suficientes, o painel usa estimativa conservadora por posição no ranking e explica isso no texto.
- **Correção da classificação de departamentos**: produtos de outros domínios (canecas, sandálias, livros, pneus, etc.) não caem mais em Eletrônicos — classificação por domínio + nome.
- **Alertas de produto em alta**: base estável por palavra-chave (não mais por item do ML) e persistida em `localStorage`, então alertas continuam valendo entre sessões.
- **Mobile mais fluido**: bloqueio do refresh nativo ao arrastar no topo (pull-to-refresh) e troca de categorias por swipe sem a "piscada" de rolagem de 1s.

### 06/08/2026 — Aba Departamentos útil e carregamento rápido

- **Aba Departamentos**:
  - Clicar no card de um departamento agora abre a aba **Produtos já filtrada** por aquele departamento, em vez de só abrir o Mercado Livre externo. O link externo virou um botão separado **"Ver no ML"**.
  - **Backfill de departamentos**: ao carregar os dados reais, o painel busca automaticamente produtos representativos para os departamentos que ficarem vazios (ex.: air fryer, bicicleta aro 29, óleo de motor), então **nenhum departamento fica com 0 produtos**.
- **Carregamento mais rápido e sem travamento**:
  - As buscas de produtos reais agora rodam **em paralelo** (pool de concorrência), reduzindo muito o tempo da primeira carga.
  - Timeout de segurança em cada chamada ao proxy (25s), para nenhuma requisição travar a página para sempre.
  - Trava contra atualizações simultâneas: clicar em "Atualizar métricas" duas vezes não duplica a carga.
  - Mensagem no carregamento avisando que a primeira carga pode levar cerca de um minuto.

### 06/08/2026 — Sazonalidade automática e correções de layout

- **Sazonalidade automática**: produtos passam a receber automaticamente os selos sazonais (🎄 Natal, 💝 Dia das Mães, 👔 Dia dos Pais, ❄️ Inverno, 🔥 Verão, 🎒 Volta às aulas, ⚽ Copa/eventos) com base em palavras-chave do nome — inclusive os produtos reais vindos da API do Mercado Livre. A classificação pode ser ajustada manualmente no cadastro do produto.
- **Correção de overflow**:
  - No gráfico *Evolução de buscas*, títulos longos são abreviados com reticências para nunca colidirem com a data lateral (nome completo aparece no tooltip).
  - Na legenda do gráfico de pizza, nomes longos truncan e os números não estouram mais o card.
- **Correção de erro de carregamento**: corrigido um bug (uso de variável antes da inicialização) que travava a renderização do painel e deixava produtos e botões indisponíveis.
- **Melhorias no relatório PDF**:
  - Rodapé atualizado para citar os dados reais da API oficial do Mercado Livre.
  - Novo resumo executivo no topo com os destaques do catálogo.
  - Selo verde "Dados reais do ML" no cabeçalho.
  - Gráfico donut de participação por departamento ao lado da tabela.
  - Paginação em A4 com linhas e títulos sem quebras órfãs.
- **Painel visual**:
  - Cards de indicadores (KPIs) com ícone e faixa de cor temática no Painel e em Oportunidades do dia.
  - Selo de última atualização mais destacado (borda verde + indicador pulsante).
  - Botão flutuante "voltar ao topo" que aparece após rolar a página.

## Documentação complementar

- `respostas-time-ti.md` — respostas para a submissão ao time de TI.
- `roteiro-demo.md` — roteiro de demonstração do painel.
