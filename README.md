**Instituto de Tecnologia e Liderança**

Engenharia de Software — Módulo: 2026.2A.ES07

**Atividade Ponderada \- Aprendizado Contínuo**  
Proposta de atualização contínua para sistemas conversacionais

**Matheus Ferreira da Silva**

02/10/2026

# **1\. INTRODUÇÃO**

Sistemas conversacionais costumam combinar um classificador de intenções, treinado sobre um conjunto de frases de exemplo, com uma base de documentos consultada para montar as respostas. Em geral, esse classificador é treinado uma única vez antes da entrada em produção, enquanto o domínio em que o sistema opera continua mudando, uma vez que surgem novos assuntos, os usuários passam a escrever de outra forma e as informações consultadas são atualizadas. Dessa forma, um modelo que não é atualizado perde qualidade de maneira silenciosa, sem que nenhum erro explícito apareça.

Esse fenômeno é conhecido como *concept drift*, ou deriva de conceito, e ocorre quando o conceito aprendido pelo modelo muda ao longo do tempo, muitas vezes por fatores que não são observados diretamente, fazendo com que ele deixe de representar a realidade em que opera (WIDMER; KUBAT, 1996). Essa mudança pode ser virtual, quando afeta apenas a forma como as entradas chegam ao sistema, ou real, quando altera a relação entre as entradas e as intenções esperadas (GAMA et al., 2014). Em um sistema conversacional, o primeiro caso aparece quando as mensagens passam a chegar por voz, com erros de transcrição que não aparecem no texto digitado, enquanto o segundo aparece quando uma pergunta antes considerada fora de escopo passa a pertencer a uma nova intenção.

Além disso, retreinar modelos de linguagem com dados novos tende a provocar o esquecimento daquilo que já havia sido aprendido (JANG et al., 2022). Uma alternativa é manter as informações em uma base externa de documentos, que pode ser atualizada sem novo treinamento (LEWIS et al., 2020), o que, por outro lado, exige cuidado para que informações antigas e novas não convivam sem distinção. Nesse sentido, atualizar um sistema conversacional não se resume a treinar o modelo novamente, mas envolve decidir o que deve ser atualizado, com qual frequência e com qual validação.

# **2\. SOLUÇÃO PROPOSTA**

A proposta separa a atualização em dois ciclos. O ciclo rápido atualiza a base de documentos sempre que uma fonte de informação é alterada, sem treinar nenhum modelo, enquanto o ciclo lento retreina o classificador de intenções, passando antes pela validação de especialistas do negócio. Essa separação parte do fato de que as informações consultadas pelo sistema mudam com muito mais frequência do que a forma como os usuários fazem seus pedidos. Dessa forma, faz mais sentido atualizar as informações diretamente na base de documentos, sem alterar o modelo, enquanto o classificador, por ser pequeno, pode ser retreinado sem custo relevante sempre que surgirem novos padrões de mensagem. Essa organização está representada na Figura 1, e as responsabilidades de cada módulo são descritas a seguir.

Figura 1 \- Arquitetura de aprendizado contínuo

----diagrama

Fonte: Desenvolvida pelo autor (2026).

**M1 \- Registro de interações e feedback:** guarda cada interação, com o texto anonimizado, a intenção prevista, a confiança do classificador, a versão do modelo e o feedback do usuário, além de marcar as perguntas fora do catálogo de intenções. Assim, o mesmo log que atende à rastreabilidade passa a servir de matéria-prima para o aprendizado.

**M2 \- Monitor de deriva:** como em produção não existe a resposta correta no momento da predição, o monitor acompanha sinais indiretos, como a queda da confiança média, o aumento de perguntas fora do catálogo e o aumento de feedbacks negativos, gerando um alerta quando esses valores se afastam do comportamento esperado. Esse mesmo acompanhamento também indica se uma nova versão do modelo piorou depois de entrar em produção.

**M3 \- Curadoria humana:** seleciona as mensagens em que o modelo teve menos certeza e agrupa as perguntas fora do catálogo por tema, para que especialistas do negócio confirmem ou corrijam as intenções. A criação de uma nova intenção continua sendo uma decisão humana, mantendo o catálogo sob controle.

**M4 \- Retreino e avaliação:** retreina o classificador com os dados antigos somados aos novos, o que evita o esquecimento sem custo relevante, e compara o resultado em dois conjuntos de teste, um fixo com as intenções que não podem piorar e outro com os exemplos novos. O modelo só segue adiante se melhorar nos exemplos novos sem piorar nas intenções antigas.

**M5 \- Implantação controlada:** registra cada versão do modelo e só a coloca em produção após aprovação do responsável pelo negócio, mantendo a versão anterior disponível para retorno caso o monitor de deriva identifique piora nos indicadores.

**M6 \- Atualização da base de conhecimento:** reprocessa apenas os documentos alterados e marca a versão anterior como não vigente, evitando que o sistema responda com uma informação desatualizada.

## **2.1 Aplicação no projeto do Metrô**

No agente de portfólio de projetos desenvolvido para o Metrô, essa proposta se encaixa de forma direta. O ciclo rápido atualizaria a base sempre que o status mensal dos projetos ou algum normativo fosse alterado no SharePoint, enquanto o ciclo lento teria o PMO como responsável pela curadoria. Além disso, a TAPI já prevê que interações fora do catálogo de intenções sejam detectadas e descartadas (COMPANHIA DO METROPOLITANO DE SÃO PAULO, 2026), e, durante o kickoff, o próprio Metrô pediu que essas perguntas também fossem registradas para identificar novas demandas, o que corresponde exatamente ao papel dos módulos M1 e M3.

# **3\. CONCLUSÃO**

Ao desenvolver esta proposta, percebi que o maior desafio do aprendizado contínuo em sistemas conversacionais não está no algoritmo, mas no processo em torno dele, uma vez que retreinar um classificador é simples, enquanto garantir dados confiáveis, validação humana e possibilidade de voltar atrás exige mais organização. Considero que o ponto mais importante do artigo estudado foi mostrar que atualizar o conhecimento dentro do próprio modelo provoca esquecimento, o que reforçou a decisão de manter as informações na base de documentos e reservar o retreino apenas para o classificador.

Em relação ao esforço de implementação, o registro de interações, o retreino e a atualização da base seriam os módulos mais simples, uma vez que costumam aproveitar estruturas existentes no sistema, como logs e o pipeline de recuperação de documentos. Por outro lado, a curadoria e a implantação controlada seriam os mais custosos, pois dependem do tempo de especialistas e de processos de aprovação. No caso do projeto do Metrô, como o MVP usa apenas dados sintéticos, o monitor de deriva só poderia ser validado por simulação, o que torna razoável implementar primeiro uma versão mínima com os módulos M1, M4 e M6 e deixar o restante para uma etapa futura.

# **REFERÊNCIAS BIBLIOGRÁFICAS**

COMPANHIA DO METROPOLITANO DE SÃO PAULO. **Termo de Abertura do Projeto Inteli (TAPI)**: Agente de IA para Portfólio de Projetos do Metrô. São Paulo: Inteli, 2026. Documento de circulação restrita.

GAMA, João *et al*. A survey on concept drift adaptation. **ACM Computing Surveys**, New York, v. 46, n. 4, p. 1-37, 2014.

JANG, Joel *et al*. Towards continual knowledge learning of language models. **arXiv preprint**, arXiv:2110.03215v4, 2022. Disponível em: https://arxiv.org/abs/2110.03215. Acesso em: 2 out. 2026.

LEWIS, Patrick *et al*. Retrieval-augmented generation for knowledge-intensive NLP tasks. *In*: CONFERENCE ON NEURAL INFORMATION PROCESSING SYSTEMS, 34., 2020, evento online. **Advances in Neural Information Processing Systems 33**. [*S. l.*]: Curran Associates, 2020. p. 9459-9474.

WIDMER, Gerhard; KUBAT, Miroslav. Learning in the presence of concept drift and hidden contexts. **Machine Learning**, [*S. l.*], v. 23, n. 1, p. 69-101, 1996.

**Observação sobre uso de Inteligência Artificial:**

A ferramenta Claude (Anthropic) foi utilizada como apoio nesta atividade para quatro finalidades específicas: adaptação da estrutura de relatório já utilizada pelo autor ao formato desta ponderada, apoio na leitura e síntese do artigo de Jang et al. (2022), revisão crítica do conteúdo e revisão da redação e da formatação das referências segundo a ABNT. As decisões de arquitetura, a divisão de responsabilidades entre os módulos e as conclusões foram avaliadas criticamente pelo autor e permanecem sob sua responsabilidade.

Caso haja qualquer dúvida sobre a veracidade do meu domínio sobre o conteúdo apresentado, coloco-me à disposição para ser questionado, arguido ou abordado da forma que os professores julgarem adequada.
