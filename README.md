



## Fluxo operacional



1. `time-driven` diário ou a cada 6 horas para gerar recomendações em lote pequeno.  

2. `onEdit` na planilha para registrar preferências, assinatura e feedback do usuário.  

3. `onFormSubmit` se você optar por cadastro via Google Forms.  

4. `doGet`/`doPost` apenas se a interface precisar de Web App, mas sem depender disso para automação principal.  



A ideia é evitar polling pesado e recalcular somente quando houver mudança ou em janelas fixas, para não estourar runtime nem URL Fetch. [developers.google](https://developers.google.com/apps-script/guides/services/quotas)



## Modelagem em texto puro



Tudo pode ficar em Sheets, com abas simples:



- `Usuarios`: id, email, nome, plataformas_assinadas, budget, tempo_disponivel, genero_favorito, sensibilidade_tematica.  

- `Catalogo`: id_filme, titulo, duracao, genero, tema_textual, plataforma, custo_extra, descricao_curta.  

- `Historico`: user_id, id_filme, data, nota, status.  

- `Recomendacoes`: user_id, data, lista_texto, justificativa_texto, custo_estimado, alternancia_texto.  



Como você pediu texto plano inclusive na autenticação, a chamada à Gemini pode usar uma chave armazenada em `PropertiesService` e enviada em header ou querystring conforme o endpoint usado, sem interface visual nem OAuth interativo no fluxo de negócio. [developers.google](https://developers.google.com/apps-script/guides/services/quotas)



## Estratégia de recomendação



A lógica deve ser híbrida: regra determinística para alternar plataformas e Gemini para enriquecer tema e justificativa. Primeiro você filtra por disponibilidade e preferências do usuário; depois monta uma sequência curta, por exemplo 3 a 5 filmes, tentando alternar plataformas entre títulos consecutivos quando isso não aumentar custo.  



Um score simples funciona bem no free tier: `score = afinidade_temática - penalidade_custo + bônus_alternância`. Assim você mantém a “fronteira dos sentidos” como coerência estética, mas força troca de plataforma quando houver equivalência razoável entre opções. [developers.google](https://developers.google.com/apps-script/guides/services/quotas)



## Gatilhos recomendados



- `onEdit`: atualiza perfil e feedback imediatamente.  

- `time-driven` 1x por dia: recalcula recomendações para todos os usuários ativos.  

- `time-driven` a cada 6 horas: apenas para usuários com alta atividade ou fila pequena.  



Como o runtime total de triggers no free tier é 90 minutos por dia, o melhor é processar em lotes pequenos, com `PropertiesService` guardando cursor de execução para continuar no próximo disparo. [developers.google](https://developers.google.com/apps-script/guides/services/quotas)



## Autenticação em texto plano



Se a exigência é “tudo em texto plano”, mantenha apenas uma chave simples no Properties do projeto e use uma função de encapsulamento como `getGeminiKey()`. Não mostre tela de login própria nem token em HTML; o usuário só interage com a planilha ou formulário. Isso reduz complexidade e evita gasto extra de runtime com autenticação de interface.  



## Limites práticos do free tier



Para não estourar o plano gratuito:



- manter execuções abaixo de 6 minutos cada.  

- limitar chamadas Gemini a poucas por lote, idealmente uma por usuário por ciclo.  

- usar texto curto nos prompts e respostas.  

- evitar reprocessar catálogo inteiro sempre; atualizar só incrementos.  



Os limites oficiais do Apps Script incluem 6 min por execução, 20 triggers por usuário/projeto e 20.000 URL Fetch/dia; triggers também entram no teto de 90 min/dia no consumer account. [developers.google](https://developers.google.com/apps-script/guides/services/quotas)



## Estrutura mínima sugerida



- `Config.gs`: constantes, chaves e parâmetros.  

- `Data.gs`: leitura e escrita nas abas.  

- `Recommender.gs`: score, alternância e montagem da sequência.  

- `Gemini.gs`: chamada textual e parsing da resposta.  

- `Triggers.gs`: criação e manutenção dos gatilhos nativos.  





