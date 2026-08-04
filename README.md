# PRONIX Radar

Radar de produtos mais buscados no Mercado Livre para análise de demanda e oportunidade. Painel de uso interno da equipe Pronix.

## Novidade: dados reais do Mercado Livre

Desde esta atualização, o painel passa a exibir **dados reais do Mercado Livre** (preço, avaliações, tendências) sem exigir login dos especialistas. Antes, os valores eram estimativas locais; agora são coletados da API oficial do ML por um servidor intermediário.

### Como funciona

- Uma **Netlify Function** (`ml-bridge`) na URL `https://sunny-paprenjak-43d157.netlify.app/.netlify/functions/ml-bridge` faz o papel de proxy seguro para a API do Mercado Livre.
- O **token de acesso** (autorização única feita pelo administrador) é guardado no **Netlify Blobs** e **renova automaticamente** a cada ~6h graças ao escopo `offline_access` — sem intervenção manual.
- O painel (`index.html` publicado em GitHub Pages) consulta o proxy com uma chave de API própria; os especialistas **não fazem login** no Mercado Livre.

### Infraestrutura

| Item | Valor |
| --- | --- |
| Painel (GitHub Pages) | `https://lucasramosseo-droid.github.io/pronix-radar/` |
| Proxy (Netlify Function) | `https://sunny-paprenjak-43d157.netlify.app/.netlify/functions/ml-bridge` |
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

Abra `index.html` em qualquer navegador (ou o painel publicado). Não requer instalação. Funciona online (para dados reais via proxy) e offline (estimativas locais).

## Documentação complementar

- `respostas-time-ti.md` — respostas para a submissão ao time de TI.
- `roteiro-demo.md` — roteiro de demonstração do painel.
