---
name: lgpd-anonimizacao-dados-pessoais
description: >-
  Identifica, anonimiza, pseudonimiza e tarja de forma definitiva dados pessoais,
  dados pessoais sensíveis e metadados ocultos em documentos (PDF, DOCX, XLSX, CSV,
  TXT, MD, JSON e imagens), em conformidade com a LGPD (Lei nº 13.709/2018),
  preservando o formato e o layout originais do arquivo.
version: 1.0.0
license: MIT
---

# LGPD — Anonimização e Proteção de Dados Pessoais

## 1. Propósito e Base Legal

Esta skill orienta o agente a sanitizar documentos antes de compartilhamento, publicação, auditoria ou uso em treinamentos, mitigando riscos de exposição indevida de dados em conformidade com a **Lei Geral de Proteção de Dados Pessoais (LGPD — Lei nº 13.709/2018)**:

- **Art. 5º, I (Dado pessoal):** informação relacionada a pessoa natural identificada ou identificável.
- **Art. 5º, II (Dado pessoal sensível):** origem racial ou étnica, convicção religiosa, opinião política, filiação a sindicato ou a organização de caráter religioso, filosófico ou político, dado referente à saúde ou à vida sexual, dado genético ou biométrico.
- **Art. 5º, III e XI / Art. 12 (Anonimização):** utilização de meios técnicos razoáveis e disponíveis no momento do tratamento, por meio dos quais um dado perde a possibilidade de associação, direta ou indireta, a um indivíduo.
- **Art. 13, § 4º (Pseudonimização):** tratamento por meio do qual um dado perde a possibilidade de associação, direta ou indireta, a um indivíduo, senão pelo uso de informação adicional mantida separadamente.

---

## 2. Escopo de Detecção

### 2.1. Proteção obrigatória (padrão)

Identifique e proteja integralmente:

1. **Identificadores diretos e indiretos de pessoas naturais:**
   - Nomes completos, sobrenomes isolados, iniciais identificáveis e assinaturas manuscritas.
   - Documentos oficiais: CPF, RG, CNH, passaporte, título de eleitor, PIS/PASEP, registro profissional (OAB, CRM, CREA, etc.).
   - Identificadores institucionais/funcionais: matrículas acadêmicas, registros funcionais, logins pessoais e nomes de usuário vinculados a pessoas reais.
   - Dados de contato e localização: e-mails pessoais ou nominais, telefones, endereços residenciais, CEP vinculado a residência e endereços IP públicos atribuídos a indivíduos.
   - Dados bancários ou financeiros vinculados a pessoa física.
2. **Dados pessoais sensíveis (Art. 5º, II):** quaisquer menções associadas a pessoa natural identificável.
3. **Metadados ocultos:** propriedades internas de arquivos (`Author`, `Creator`, `Producer`, `Title`, `Subject`, `Keywords`, histórico de revisões, comentários, propriedades XMP e metadados EXIF de imagens).

### 2.2. Proteção sob demanda (opcional)

Quando solicitado explicitamente pelo usuário — ou sinalizado ao final como sugestão de revisão opcional:

- Razão social, nome fantasia, siglas, domínios internos, certificados digitais e nomes de bancos/arquivos derivados da organização avaliada.
- Referências institucionais específicas no corpo do texto (ex.: instituição de ensino ou cliente).
- Links públicos externos (formulários, documentos compartilhados) e identificadores técnicos de nuvem (Subscription ID, Tenant ID, Virtual Network ID, Resource Group, IPs públicos de laboratório).

---

## 3. Regras de Varredura e Decisão

1. **Decomposição em tokens parciais:**
   - Para cada nome completo identificado, decomponha-o em tokens individuais (nomes e sobrenomes relevantes, excluindo preposições curtas como `de`, `da`, `do`, `dos`, `das`, `e`).
   - Execute a busca de forma _case-insensitive_ e normalizada (com e sem acentuação) para capturar ocorrências isoladas em legendas, créditos de figuras, tabelas, rodapés, cabeçalhos, logs e substrings (como prefixos de e-mail ou nomes de usuário).
2. **Regra _Fail-Safe_ (Dados fictícios vs. reais):**
   - Contas e nomes comprovadamente fictícios de ambiente de laboratório (ex.: `Cliente Exemplo 1`, `teste@exemplo.com`, faixas RFC 5737) podem ser preservados apenas quando **não** houver coincidência com nenhum dado pessoal real do documento.
   - Havendo qualquer dúvida razoável ou coincidência parcial com um dado real, **a proteção prevalece**.
3. **Preservação de integridade técnica:**
   - Não remova conteúdo técnico neutro (comandos, portas, protocolos, métricas, siglas técnicas) nem danifique textos vizinhos à área protegida.

---

## 4. Estratégias de Proteção por Formato

Mantenha sempre o **formato original** e o **layout visual** do arquivo enviado pelo usuário, salvo solicitação expressa em contrário.

### 4.1. Arquivos PDF (`.pdf`)

- **Tarjamento definitivo:** localize as coordenadas exatas de cada ocorrência (incluindo quebras de linha) e aplique tarjas pretas sólidas (`#000000`) com pequena margem de segurança para impedir vazamento de bordas de glifos.
- **Rasterização obrigatória das páginas alteradas:** converta toda página modificada em imagem de alta resolução (**180 a 200 DPI**) e reconstrua o PDF substituindo a página original pela versão rasterizada. Isso elimina completamente a camada de texto selecionável por baixo da tarja (evitando o "falso tarjamento").
- **Limpeza integral de metadados:** remova todos os campos de propriedades do documento e streams XMP antes de salvar o arquivo final.

### 4.2. Imagens isoladas ou embutidas (`.png`, `.jpg`, capturas de tela em PDFs)

- Inspecione capturas de tela (painéis em nuvem, terminais, navegadores, logs, formulários) em busca de nomes, e-mails, matrículas, CPFs, telefones, URLs ou IDs pessoais.
- Aplique tarja preta sólida irreversível diretamente sobre os pixels afetados.
- **Limitação de ambiente sem OCR:** caso o ambiente de execução não disponha de motor OCR para localizar automaticamente coordenadas dentro de imagens rasterizadas em documentos extensos, inspecione visualmente os trechos críticos e **liste explicitamente** no relatório de auditoria as páginas e figuras que exigem revisão ou tarjamento manual.

### 4.3. Documentos editáveis (`.docx`)

- Aplique **pseudonimização consistente por rótulos tipados** (`[NOME_1]`, `[CPF_1]`, `[EMAIL_1]`, `[TELEFONE_1]`, `[ENDERECO_1]`), preservando a correlação referencial ao longo do texto (o mesmo titular recebe sempre o mesmo identificador).
- Preserve estilos, fontes, tabelas, cabeçalhos e rodapés.
- Limpe todas as propriedades (`core_properties`) do documento.

### 4.4. Planilhas e dados tabulares (`.xlsx`, `.csv`)

- Substitua apenas as células contendo dados pessoais utilizando pseudonimização tipada consistente.
- Preserve nomes de abas, cabeçalhos, delimitadores, codificação, formatação numérica e fórmulas.
- Em arquivos `.xlsx`, limpe também os metadados da pasta de trabalho.

### 4.5. Arquivos de texto e dados estruturados (`.txt`, `.md`, `.json`)

- Aplique pseudonimização tipada mantendo a sintaxe rigorosamente válida (JSON válido, indentação preservada, marcação Markdown intacta).

### 4.6. Modos alternativos sob demanda

Quando solicitado pelo usuário, adapte a substituição textual para:

- **Anonimização irreversível genérica:** `[DADO_OCULTADO]` (sem correlação entre ocorrências).
- **Mascaramento parcial:** preservação mínima de formato (ex.: `***.***.***-XX`, `c***@dominio.com`).

---

## 5. Renomeação Opcional por Hash SHA-256

Sempre que o usuário solicitar renomeação por hash único ou hash baseado no conteúdo:

1. Calcule o digest **SHA-256** (hexadecimal em minúsculas) sobre o conteúdo binário integral do arquivo **já sanitizado**.
2. Renomeie o arquivo para `<sha256>.<extensao_original>`.
3. Valide o hash novamente após a gravação em disco e informe o valor e o tamanho em bytes no resumo.

---

## 6. Protocolo Obrigatório de Validação Pós-Processamento

Antes de entregar o arquivo final, execute todas as verificações abaixo:

1. **Varredura residual de texto:** reabra o arquivo gerado e pesquise (de forma normalizada e _case-insensitive_) por todos os nomes completos, tokens parciais, CPFs, e-mails, telefones e termos alvo -> o resultado deve ser **0 vazamentos residuais**.
2. **Verificação da camada de texto (PDF):** confirme que todas as páginas alteradas possuem **0 caracteres extraíveis** após a rasterização e que todos os metadados do PDF estão vazios.
3. **Verificação visual (_spot-check_):** inspecione visualmente amostras das páginas/imagens tarjadas para confirmar cobertura sólida completa, ausência de caracteres parcialmente expostos e integridade do texto adjacente.
4. **Integridade do arquivo:** confirme que o arquivo abre corretamente, mantém o número original de páginas/abas/registros e que a estrutura sintática (quando aplicável) está válida.

---

## 7. Formato da Resposta e Sigilo no Relatório de Auditoria

### Regra de sigilo absoluto

**NUNCA** repita, transcreva ou exiba nenhum dado pessoal real removido (nomes, CPFs, e-mails, matrículas, telefones ou endereços) na resposta ao usuário, na tabela de auditoria ou no nome do arquivo gerado.

### Estrutura obrigatória da entrega

1. **Link/anexo do arquivo sanitizado.**
2. **Resumo de Auditoria (LGPD — Lei nº 13.709/2018):**
   - Nome do arquivo gerado, número de páginas/registros e método aplicado.
   - Tabela consolidada contendo:
     - `Categoria de Dado Pessoal` (com referência ao artigo da LGPD quando aplicável)
     - `Origem` (texto, tabela, metadados, imagem embutida)
     - `Ocorrências Protegidas` (contagem numérica)
     - `Páginas / Seções Afetadas`
3. **Validação Executada:** resultados objetivos da varredura residual, camada de texto, _spot-check_ visual e integridade (incluindo SHA-256 se utilizado).
4. **Alertas ou Revisão Opcional (quando aplicável):** aponte de forma genérica (sem expor valores sensíveis) eventuais capturas de tela sem OCR, links públicos ou identificadores técnicos que o usuário possa querer tarjar em uma segunda passada.
