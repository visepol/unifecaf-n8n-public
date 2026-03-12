# Parte Teórica – Análise e Discussão
### Disciplina: Fundamentos de IA com foco em IA Generativa

---

## 1. Contextualização do Desafio

O processo de triagem de currículos é uma das tarefas mais repetitivas e custosas para equipes de Recursos Humanos. Em empresas de tecnologia, especialmente fintechs com alto volume de candidaturas para vagas técnicas, um recrutador pode receber dezenas de currículos por dia para uma única vaga. Analisar manualmente cada documento, verificar requisitos obrigatórios como experiência com tecnologias específicas e anos de atuação profissional, e ainda redigir respostas individuais para cada candidato consome horas de trabalho que poderiam ser destinadas a atividades de maior valor estratégico.

O desafio proposto foi automatizar integralmente esse fluxo: desde o recebimento do currículo por e-mail até o envio da resposta ao candidato e o registro do resultado em uma planilha de controle  sem nenhuma intervenção humana manual. A solução precisava ser capaz de interpretar o conteúdo de documentos PDF não estruturados, avaliar critérios técnicos específicos e comunicar a decisão de forma clara e profissional.

---

## 2. Justificativa para o Uso de IA Generativa na Solução

A triagem de currículos é um problema que envolve linguagem natural não estruturada. Currículos não seguem um padrão único  cada candidato organiza seu documento de forma diferente, utiliza terminologias variadas e descreve suas experiências com diferentes níveis de detalhe. Abordagens tradicionais baseadas em palavras-chave ou expressões regulares falham nesse contexto, pois são frágeis a variações de escrita e não capturam o contexto semântico das informações.

A IA Generativa, especificamente os Modelos de Linguagem de Grande Escala (LLMs), resolve esse problema com naturalidade: esses modelos foram treinados em enormes volumes de texto humano e são capazes de compreender o significado de um currículo, identificar experiências profissionais relevantes, inferir anos de atuação e avaliar se o perfil atende aos requisitos da vaga  mesmo que o candidato não utilize exatamente as palavras-chave esperadas.

Além disso, o uso de IA Generativa permitiu que a solução fosse construída sem nenhuma linha de código complexo para a parte de análise, tornando-a acessível, configurável e facilmente adaptável para diferentes perfis de vagas apenas alterando o prompt.

---

## 3. Modelo LLM Utilizado

O modelo utilizado foi o **Google Gemini 2.0 Flash Lite**, acessado via API através da integração nativa do n8n com o Google Gemini (PaLM API).

O Gemini é um modelo multimodal desenvolvido pelo Google DeepMind, treinado em um vasto conjunto de dados textuais e capaz de realizar tarefas de compreensão de linguagem natural, extração de informações, raciocínio e geração de texto estruturado. A variante **Flash Lite** foi escolhida por oferecer um excelente equilíbrio entre capacidade de análise e velocidade de resposta, sendo adequada para processamento automatizado em tempo real.

Características relevantes para este projeto:
- **Compreensão contextual**: capaz de interpretar experiências profissionais descritas em linguagem natural
- **Geração estruturada**: instrução para retornar respostas no formato JSON facilita a integração com etapas posteriores do workflow
- **Custo-benefício**: modelo eficiente para tarefas de classificação e extração de informação
- **Disponibilidade via API**: integração direta com n8n sem necessidade de infraestrutura adicional

---

## 4. Descrição de Como o Prompt Foi Elaborado

O prompt foi construído seguindo princípios de **prompt engineering** para garantir respostas consistentes, estruturadas e úteis para automação.

### Prompt utilizado:

```
Você é um recrutador de uma fintech. Analise o currículo abaixo e determine
se o candidato atende aos requisitos da vaga.

REQUISITOS OBRIGATÓRIOS:
- Desenvolvedor Senior (5+ anos de experiência)
- Experiência com Node.js
- Experiência com AWS

Responda APENAS no formato JSON abaixo, sem texto adicional, sem markdown:
{
  "aprovado": true ou false,
  "nome_candidato": "nome completo extraído do currículo",
  "justificativa": "explique em 1 ou 2 frases por que foi aprovado ou rejeitado, citando os critérios"
}

CURRÍCULO:
{{ $json.text }}
```

### Decisões de design do prompt:

**Persona definida**: instruir o modelo a agir como "recrutador de uma fintech" estabelece um contexto profissional e direciona o tom e o critério de avaliação.

**Critérios explícitos e objetivos**: os requisitos obrigatórios foram listados de forma clara e mensurável, evitando ambiguidade na avaliação.

**Formato de saída forçado**: ao exigir resposta exclusivamente em JSON, elimina-se a variabilidade de formato que poderia quebrar as etapas seguintes do workflow. A instrução "sem texto adicional, sem markdown" previne que o modelo adicione explicações ou blocos de código ao redor do JSON.

**Campos estratégicos**: os três campos do JSON foram escolhidos para atender necessidades específicas  `aprovado` para o nó de decisão, `nome_candidato` para personalização dos e-mails e registro na planilha, e `justificativa` para transparência e auditoria da decisão.

**Injeção dinâmica do conteúdo**: o placeholder `{{ $json.text }}` é substituído em tempo de execução pelo texto extraído do PDF, tornando o prompt reutilizável para qualquer currículo.

---

## 5. Benefícios Percebidos e Desafios Enfrentados

### Benefícios

**Redução drástica do tempo de triagem**: o processo que poderia levar horas de análise manual é executado em segundos por currículo, desde o recebimento do e-mail até o envio da resposta ao candidato.

**Consistência na avaliação**: diferente da análise humana, que pode variar conforme o estado de atenção do recrutador, o modelo aplica os mesmos critérios de forma uniforme para todos os candidatos.

**Escalabilidade**: a solução processa qualquer volume de candidaturas sem perda de qualidade ou necessidade de aumento de equipe.

**Rastreabilidade**: todos os resultados são registrados automaticamente em uma planilha Google Sheets com data, nome, e-mail, status e justificativa da decisão, criando um histórico auditável de todo o processo seletivo.

**Comunicação imediata**: o candidato recebe uma resposta assim que seu currículo é analisado, melhorando significativamente a experiência com a empresa.

**Baixo custo de manutenção**: alterar os critérios da vaga requer apenas a edição do prompt, sem necessidade de reprogramar a solução.

### Desafios Enfrentados

**Integração OAuth com ambiente de produção**: as credenciais OAuth do Google (Gmail e Google Sheets) são vinculadas a URLs de callback específicas, o que gerou dificuldades ao migrar a solução do ambiente local para o servidor em nuvem, já que o Google não aceita endereços IP diretamente como URI de redirecionamento válido.

**Gerenciamento de estado do trigger**: o Gmail Trigger do n8n interrompe o polling automático após erros de execução, exigindo atenção para que emails sem PDF não interrompam o fluxo  problema resolvido com a adição de um nó de filtro antes do processamento.

---

## 6. Limites Éticos e de Segurança

### Viés da IA

Modelos de linguagem como o Gemini são treinados em dados históricos da internet, que podem conter vieses relacionados a gênero, etnia, nacionalidade ou formação acadêmica. No contexto desta solução, existe o risco de que o modelo avalie currículos de forma diferente dependendo de características do candidato que não deveriam ser critérios de seleção  como o nome (que pode indicar gênero ou origem étnica) ou o nome da instituição de ensino.

Para mitigar esse risco, o prompt foi desenhado para focar exclusivamente em critérios técnicos objetivos e mensuráveis (anos de experiência, tecnologias específicas), sem solicitar avaliação de aspectos subjetivos. Ainda assim, recomenda-se que decisões finais de contratação sempre envolvam revisão humana.

### Vazamento de Dados e LGPD

Currículos contêm dados pessoais sensíveis: nome completo, endereço, CPF em alguns casos, histórico profissional e informações de contato. Ao enviar esses dados para a API do Google Gemini, os dados trafegam por servidores externos à organização.

Sob a ótica da **Lei Geral de Proteção de Dados (LGPD)**, isso configura tratamento de dados pessoais por terceiros, exigindo:
- **Base legal**: o candidato deve ter consentido com o tratamento de seus dados ao submeter o currículo
- **Acordo de processamento de dados**: a empresa deve verificar os termos de uso da API do Google para confirmar que os dados não são utilizados para treinamento de modelos sem consentimento
- **Minimização de dados**: o prompt solicita apenas as informações estritamente necessárias para a decisão de triagem
- **Retenção limitada**: os dados armazenados na planilha Google Sheets devem ter política de retenção definida

### Transparência Algorítmica

O candidato recebe apenas a decisão final (aprovado ou rejeitado) com uma justificativa geral. A solução não expõe que a triagem foi realizada por IA, o que pode ser questionável do ponto de vista ético. Boas práticas recomendam informar os candidatos quando processos seletivos utilizam triagem automatizada por inteligência artificial, garantindo transparência e possibilidade de contestação da decisão.

### Disponibilidade e Dependência de Terceiros

A solução depende da disponibilidade de múltiplos serviços externos (Gmail API, Gemini API, Google Sheets API). Falhas ou mudanças de política nesses serviços podem interromper o processo seletivo sem aviso prévio, exigindo monitoramento contínuo e plano de contingência.

---

*Documento elaborado com base na solução de triagem automatizada de currículos desenvolvida com n8n, Google Gemini e Google Workspace.*
