# Respostas — Pronix Radar (submissão ao time de TI)

Ferramenta de uso interno da equipe Pronix para análise de demanda e oportunidade de produtos no Mercado Livre.

---

## 1. Descrição da tarefa/problema resolvido

A equipe precisa decidir **quais produtos anunciar ou investir em estoque** no Mercado Livre, entendendo rapidamente o volume de buscas, a tendência e a concorrência de cada item. Antes, essa análise era manual e demorada: consultar tendências, comparar preços, estimar margem e calcular a viabilidade de cada produto.

O Pronix Radar centraliza isso em um painel único: monitora **36 produtos em 8 departamentos** (≈ 3,3 milhões de buscas/mês estimadas), calcula automaticamente para cada um o **score de oportunidade (0–100)**, a **margem**, o **lucro unitário**, o **lucro mensal estimado** e a **quantidade de estoque sugerida**, e reúne os melhores achados na aba **Oportunidades do dia**. Também gera **relatórios em PDF e CSV** para registro e compartilhamento.

## 2. Ferramenta(s) de IA utilizada(s)

Não há APIs pagas nem serviços externos de IA. A "inteligência" é um **motor de análise e estimativa embutido no próprio arquivo HTML**, executado localmente no navegador do colaborador. Ele combina:

- Regras de negócio determinísticas para score de oportunidade (volume de buscas, tendência e concorrência);
- Modelo de estimativa de conversão, margem e lucro por concorrência;
- Série histórica de 30 dias gerada por algoritmo determinístico (seed local) calibrado aos totais reais dos dados de base;
- Simulação de "análise com IA" por produto com as mesmas regras, para fins de apresentação e validação da metodologia.

**Atualização recente:** o painel passou a consumir **dados reais da API oficial do Mercado Livre** (preço, avaliações e tendências) via uma Netlify Function que atua como proxy seguro. O token de acesso é renovado automaticamente (`offline_access`), os especialistas não precisam de login e os valores exibidos vêm de dados verdadeiros do ML em vez de estimativas locais. Quando o servidor está indisponível, o painel mantém as estimativas locais como fallback.

**Hospedagem:** o painel agora é publicado no **Netlify** em `https://pronix-radar.netlify.app/` (o antigo endereço do GitHub Pages redireciona para lá). Como o painel e o proxy estão no mesmo domínio, a conexão com o ML é estável e sem bloqueios do navegador.

## 3. Estimativa de tempo economizado e frequência de uso

- **Tempo por execução**: a análise completa dos 36 produtos (buscas, tendência, concorrência, margem, lucro, quantidade sugerida e ranking de oportunidades) sai em **segundos**, com um clique em "Atualizar análise".
- **Tempo economizado**: estimativa de **20 a 30 minutos por produto** quando feito manualmente (levantar tendência + preço médio + estimar margem). Em uma rodada de 10 produtos, o ganho é de aproximadamente **3 a 5 horas**.
- **Frequência de uso**: **diária** (consulta rápida do ranking de oportunidades) e **semanal** (revisão do portfólio e geração de relatórios).

## 4. Impacto e colaboradores beneficiados

- **Abrangência**: começa de **uso individual** do colaborador, mas a solução é um arquivo único e portátil (HTML), podendo ser **replicada para todo o setor** sem custo nem instalação.
- **Colaboradores beneficiados**: estimativa de **1 a 6 colaboradores** de cada equipe de especialistas que participam da decisão de catálogo e compra de estoque, com potencial de escalar para o setor inteiro (envio do arquivo ou publicação em pasta compartilhada).

## 5. Print de tela ou vídeo curto

- Vídeo curto disponível: gravação de tela (~1 min) seguindo o roteiro de demonstração em `roteiro-demo.md`, mostrando Painel → Oportunidades do dia → catálogo com margem/lucro → relatório PDF/CSV.
- Como reproduzir: abrir `https://pronix-radar.netlify.app/` em qualquer navegador (Chrome/Edge/Firefox). O chip verde no cabeçalho indica que o Mercado Livre está conectado com dados reais.

## 6. Breve explicação de como a IA foi aplicada e qual o ganho gerado

A IA (motor de análise local) foi aplicada para **transformar dados brutos de busca em decisões prontas de negócio**. Em vez de o colaborador cruzar manualmente volume de buscas, tendência, concorrência, preço e margem, o motor:

1. Calcula um **score de oportunidade** ponderado (50% volume de buscas, 30% tendência, 20% concorrência);
2. Estima **conversão → vendas → lucro mensal** a partir da concorrência e do custo de aquisição informado;
3. Sugere **quantidade de estoque** e filtra as **oportunidades do dia** (score ≥ 60 e margem ≥ 15%);
4. Emite **alertas** quando um produto sobe ≥ 15% nas buscas, para não perder o timing de reposição.

**Ganho gerado**: decisões mais rápidas, com critério objetivo e repetível no lugar de estimativa subjetiva; economia de horas por semana; e um histórico/relatório padronizado para justificar escolhas de catálogo. O ranking atual mostra **11 oportunidades do dia**, com **lucro mensal estimado somado de R$ 1.791.943** sobre os produtos elegíveis.
