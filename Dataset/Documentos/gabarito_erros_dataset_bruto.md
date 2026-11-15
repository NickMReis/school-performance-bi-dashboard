# Gabarito — problemas injetados no dataset bruto

**Uso:** documento de referência do orientador. O arquivo `desempenho_escolar_7ano_2025_BRUTO.xlsx` (e o `notas_7ano_2025_BRUTO.csv`) contém, de forma proposital, os problemas listados abaixo. Após o tratamento, o resultado deve convergir para o dataset limpo `desempenho_escolar_7ano_2025.xlsx`.

- Registros na aba `notas_export`: **483** (sendo 15 excedentes por duplicidade)
- Registros esperados após a limpeza: **468** (78 alunos × 6 disciplinas)
- Registros na aba `alunos_export`: **78** | esperado após limpeza: **75**

| # | Categoria | Problema | Ocorrências | Tratamento esperado |
|---|---|---|---|---|
| 1 | Valores ausentes | Notas bimestrais em branco | 18 | Decidir entre imputação (média do aluno/disciplina) ou exclusão; justificar no texto. |
| 2 | Valores ausentes | Campo FALTAS em branco | 6 | Imputar 0 ou marcar como não informado; impacta o cálculo de frequência. |
| 3 | Placeholder textual | Notas preenchidas com "N/A", "-", "sem registro" | 5 | Converter para nulo antes da conversão de tipo. |
| 4 | Tipo inconsistente | Notas gravadas como texto com vírgula decimal (ex.: 7,5) | 8 | Substituir vírgula por ponto e converter para numérico. |
| 5 | Fora do domínio | Notas fora do intervalo 0–10 (55; 87; -2; 100; 12,5) | 5 | Provável erro de digitação (55 = 5,5). Regra de validação 0 <= nota <= 10. |
| 6 | Inconsistência lógica | FALTAS maior que AULAS previstas | 5 | Regra: faltas <= aulas. Tratar como erro de registro. |
| 7 | Inconsistência lógica | FALTAS com valor negativo | 3 | Regra: faltas >= 0. |
| 8 | Categoria inconsistente | Nome da disciplina com variações de caixa, acento e abreviação (MATEMATICA, matemática, MAT., Língua Portuguesa...) | 70 | Padronizar com tabela de-para; sem isso o agrupamento por disciplina se fragmenta. |
| 9 | Categoria inconsistente | Turma com espaços, prefixo "Turma ", sufixo ".0" e código alternativo | 45 | Extrair apenas o código numérico (701/702/703) via TRIM e regex. |
| 10 | Categoria inconsistente | Campo SITUACAO com variações de escrita e 4 valores nulos | 64 | Padronizar categorias; recalcular a situação a partir de média e frequência. |
| 11 | Texto não padronizado | Nomes em caixa alta/baixa, com espaços à esquerda, à direita ou duplicados | 60 | Aplicar TRIM, remoção de espaços duplos e capitalização; nunca usar o nome como chave. |
| 12 | Duplicidade | Registros integralmente duplicados | 12 | Remover por chave composta Matrícula + Disciplina. |
| 13 | Duplicidade | Mesmo aluno com segunda matrícula (sufixo +900000, OBS=REMATRICULA) | 3 | Duplicidade de cadastro: identificar por nome + data de nascimento e consolidar. |
| 14 | Tipo inconsistente | Matrícula gravada como texto em parte das linhas | 35 | Converter a chave para o mesmo tipo antes do relacionamento, senão o join falha. |
| 15 | Estrutura do arquivo | Cabeçalho real na linha 4 (título, subtítulo e linha vazia acima) e linha de totais no rodapé | 1 | No Power Query: 'Remover linhas superiores' (3) + 'Promover cabeçalhos' + remover rodapé. |
| 16 | Integridade referencial | Alunos presentes em notas_export sem registro em alunos_export | 3 | Detectar com LEFT JOIN; decidir entre excluir ou completar o cadastro. |
| 17 | Duplicidade | Alunos repetidos no cadastro | 2 | Remover duplicatas pela matrícula antes do relacionamento. |
| 18 | Integridade referencial | Aluno no cadastro sem nenhuma nota (matrícula 20250999) | 1 | Provável transferência; documentar o critério de exclusão. |
| 19 | Formato de data | Datas de nascimento em 4 formatos (dd/mm/aaaa, aaaa-mm-dd, data real, dd-mm-aa) | 4 | Converter tudo para data com tratamento de múltiplos formatos. |
| 20 | Categoria inconsistente | Sexo como M/m/Masculino/MASC e série como "7º ano"/"7 ano"/"7o ano"/"7" | 2 | Padronizar domínios categóricos. |

## Verificações de qualidade sugeridas ao final do ETL

1. Contagem final: 468 registros na tabela fato, referentes a 78 alunos — dos quais 3 não constam no cadastro e 75 possuem cadastro válido.
2. Toda matrícula da tabela fato existe na dimensão de alunos.
3. Cada aluno possui exatamente 6 disciplinas.
4. Todas as notas no intervalo de 0 a 10.
5. `0 <= faltas <= aulas previstas` em todas as linhas.
6. Domínio de disciplinas com exatamente 6 valores distintos.
7. Domínio de turmas com exatamente 3 valores distintos.
8. Situação recalculada a partir de média >= 6,0 e frequência >= 75%.

## Sugestão de encadeamento para o capítulo de Metodologia

Extração (leitura do export bruto) → Transformação (remoção de cabeçalho/rodapé, padronização de domínios, conversão de tipos, tratamento de nulos, remoção de duplicatas, validação de regras de negócio) → Carga (modelo dimensional: fato Notas + dimensões Aluno, Disciplina, Turma) → Visualização. Recomenda-se registrar o antes/depois de cada etapa, com contagem de registros, pois isso costuma render boa discussão na banca.