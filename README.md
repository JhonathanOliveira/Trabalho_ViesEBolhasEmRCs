# Viés, Equidade e Mitigação de Bolhas em Sistemas de Recomendação
Disciplina: SCC0284 – Sistemas de Recomendação

| Nome | NUSP |
| -------- | ----- |
| Jhonathan Oliveira Alves | 11838116 |
| Vinícius Morotti | 15491876 |

## Resumo
Este tutorial aprofunda os aspectos éticos e técnicos dos Sistemas de Recomendação (SRs), focando nos fenômenos do Viés Algorítmico, da falta de Equidade e da Bolha de Filtragem. Apresentamos a fundamentação conceitual desses problemas e, em complemento ao material da disciplina, detalhamos uma estratégia avançada de pós-processamento para a mitigação das Bolhas: o algoritmo Maximal Marginal Relevance (MMR). O MMR busca ativamente otimizar a diversidade da lista de recomendação, equilibrando a relevância predita para o usuário com a não-redundância do conteúdo.

Link para a videoaula: https://www.youtube.com/watch?v=fWLXjqBXDiA

Link para as avaliações da feira: https://docs.google.com/spreadsheets/d/1aKc9K0jyI1rd2DujS-ulRNJtpnZkQXyMxS4kCIpAOTY/edit?usp=sharing

## Introdução
Sistemas de Recomendação (SRs) são algoritmos de filtragem de informação projetados para predizer a preferência de um usuário por um item (produto, filme, notícia, etc.) não consumido anteriormente. Eles são pilares da economia digital (Netflix, Spotify, Amazon), operando majoritariamente por meio de Filtros Colaborativos ou Filtros Baseados em Conteúdo.
Apesar de sua utilidade, a forma como os SRs são treinados e avaliados (focando em métricas como acurácia e Root Mean Square Error - RMSE) tem gerado consequências sistêmicas negativas que afetam a justiça social e a experiência informacional do usuário.

## Viés Algorítmico e Equidade
### O Ciclo de Feedback e o Viés Algorítmico
O Viés Algorítmico em SRs não é gerado por uma intenção maliciosa, mas pela natureza observacional da coleta de dados. Conforme detalhado na teoria de feedback loop:
- O Modelo (algoritmo) gera uma lista de recomendações.
- O Usuário interage (clica, avalia) apenas com os itens expostos nessa lista.
- O Dado Coletado (observado) é, portanto, enviesado pela própria recomendação.
- O Modelo é retreinado com esse dado enviesado, reforçando os padrões anteriores.  

Este ciclo de auto-reforço leva a desvios (bias) significativos, sendo o mais comum o Viés de Popularidade: itens já populares são mais expostos, recebem mais interações e se tornam ainda mais prováveis de serem recomendados, marginalizando o conteúdo novo ou de nicho ("the rich get richer").

### A Questão da Equidade (Fairness)
A Equidade é o princípio de justiça na computação. Em SRs, exige que o sistema não amplifique ou perpetue injustiças sociais pré-existentes ou criadas pelo algoritmo. A falta de equidade se manifesta quando:
- Grupos de Usuários (ex: minorias) recebem recomendações de qualidade inferior ou menos diversas.
- Grupos de Itens (ex: filmes dirigidos por mulheres, notícias de nicho) têm sua visibilidade suprimida, mesmo que fossem relevantes para o usuário.

Exemplo Ilustrativo (Caso Amazon): O sistema automatizado de recrutamento da Amazon foi treinado em currículos históricos, onde havia predominância de candidatos homens em cargos técnicos. O algoritmo "aprendeu" que a presença de termos femininos (ex: "capitã do clube de xadrez feminino") era uma característica negativa, penalizando candidatas qualificadas. Este é um exemplo de viés que afeta a equidade de oportunidades.

## A Bolha de Filtragem (Filter Bubble)
A Bolha de Filtragem é uma consequência direta da otimização excessiva da relevância preditiva e do viés de popularidade, é um estado de isolamento informacional onde um usuário é exposto predominantemente a conteúdo que espelha seus gostos, crenças e comportamentos prévios.  

Acontece quando o algoritmo se torna excessivamente eficaz em prever o clique (relevância), mas ineficaz em promover a descoberta (novidade) ou a diversidade (variedade). O SR passa a gerar listas redundantes( itens altamente similares entre si).

### Implicações Éticas e Sociais
A busca incessante por relevância e acurácia leva o sistema a criar listas de recomendação altamente redundantes, expondo o usuário apenas a conteúdo que confirma seus gostos ou crenças prévias. Este fenômeno é um risco sociopolítico, pois restringe a diversidade de perspectivas, dificulta o aprendizado de novos tópicos e pode contribuir para a polarização social ao isolar o indivíduo em uma câmara de eco.

## Solução para Mitigação de Bolhas: Maximal Marginal Relevance (MMR)
Para combater a Bolha de Filtragem, é necessário incorporar métricas de Diversidade e Novidade no processo de recomendação, geralmente na etapa de Re-ranking.
O Maximal Marginal Relevance (MMR) é uma técnica de re-ranking (pós-processamento) que visa equilibrar o trade-off entre Relevância e Diversidade/Não-Redundância na lista final. Diferentemente de abordagens que corrigem o viés nos dados de entrada (pré-processamento) ou no treinamento do modelo, o MMR atua diretamente na lista Top-N gerada pelo modelo base.

O MMR seleciona sequencialmente o próximo item ($i$) do conjunto de candidatos restantes ($R$) que maximiza a seguinte função:  

$$\text{MMR}(i) = \lambda \cdot \text{Relevância}(i) - (1 - \lambda) \cdot \text{MaxSimilaridade}(i, S)$$  

Onde:
- $\text{Relevância}(i)$: A pontuação predita pelo modelo de base (noo caso, o SVD) para o item $i$ e o usuário, refletindo o gosto.
- $\text{MaxSimilaridade}(i, S)$: A maior similaridade (no caso, similaridade de cosseno entre os fatores latentes do SVD) entre o item $i$ e qualquer item $j$ que já foi selecionado para a lista final ($S$).
- $\lambda$: É o parâmetro de ponderação, $\lambda \in [0, 1]$, que controla o peso dado à relevância versus a diversidade:
- $\lambda \to 1$: O sistema se comporta como um Top-N padrão, priorizando a relevância.
- $\lambda \to 0$: O sistema prioriza itens com baixa similaridade, maximizando a diversidade, mas potencialmente sacrificando a relevância.

Ao subtrair o termo $\text{MaxSimilaridade}$, o MMR penaliza itens que são muito parecidos com o que já foi selecionado. Isso garante que a lista final seja composta por itens que não apenas o usuário gostará, mas que também representem uma ampla variedade de features ou categorias de conteúdo, combatendo diretamente a redundância que caracteriza a Bolha de Filtragem.
No código de demonstração em Python (Jupyter Notebook), a eficácia do MMR é comprovada pela métrica Diversidade Intra-Lista (ou Média de Dissimilaridade), que mede a dissimilaridade média entre todos os pares de itens recomendados. A aplicação do MMR com um $\lambda$ balanceado resulta em uma pontuação de Diversidade significativamente maior em comparação com a lista baseline (puramente relevante), confirmando a capacidade da técnica de promover a variedade sem anular o gosto do usuário.

## Implicações Éticas e Conclusão
Os Sistemas de Recomendação são ferramentas poderosas que demandam responsabilidade e atenção a objetivos que transcendem a acurácia. A discussão sobre Viés, Equidade e Bolha de Filtragem é central para a Engenharia de Sistemas de Recomendação de próxima geração.
A incorporação de técnicas como o Maximal Marginal Relevance (MMR) sinaliza a mudança de paradigma de uma otimização unicamente focada na acurácia para uma otimização multicritério, onde a diversidade e a equidade são parâmetros de design tão importantes quanto o desempenho preditivo. Ao educar o público e implementar soluções como o MMR, podemos capacitar usuários a serem consumidores críticos e desenvolvedores a construir algoritmos que promovam uma sociedade mais informada e equitativa.
