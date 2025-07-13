🔁 Refinamento da Lógica do Quiz com Sistema de Avaliação e Rankings
🎯 Objetivo Geral do Sistema
Criar uma aplicação de quiz gamificado com:

Categorias e níveis de dificuldade.

Pontuação por acerto.

Evolução de nível com base na pontuação acumulada.

Rankings (geral, por categoria, mensal e anual).

Dashboard de desempenho diário/semanal.

Sistema de detecção de abandono e limite de tempo de 20 segundos por pergunta.

🧠 Proposta Otimizada de Modelagem de Dados (Tabelas)
✅ 1. usuarios
Já está bem definida. Adicione apenas data_cadastro e nivel_atual como campos importantes.

sql
Copiar
Editar
ALTER TABLE usuarios ADD COLUMN data_cadastro TIMESTAMP DEFAULT now();
ALTER TABLE usuarios ADD COLUMN nivel_atual VARCHAR(50) DEFAULT 'Iniciante';
✅ 2. categorias
Para armazenar categorias temáticas.

sql
Copiar
Editar
CREATE TABLE categorias (
  id_categoria SERIAL PRIMARY KEY,
  nome_categoria VARCHAR(100) NOT NULL UNIQUE
);
✅ 3. niveis
Tabela opcional para mapear a progressão.

sql
Copiar
Editar
CREATE TABLE niveis (
  id_nivel SERIAL PRIMARY KEY,
  nome_nivel VARCHAR(50), -- Básico, Intermediário, Avançado
  pontos_minimos INT
);
✅ 4. perguntas
Melhore com campos de múltiplas opções e dificuldade.

sql
Copiar
Editar
CREATE TABLE perguntas (
  id_pergunta SERIAL PRIMARY KEY,
  id_categoria INT REFERENCES categorias(id_categoria),
  texto_pergunta TEXT NOT NULL,
  opcao_a TEXT,
  opcao_b TEXT,
  opcao_c TEXT,
  opcao_d TEXT,
  resposta_correta CHAR(1),
  nivel_dificuldade VARCHAR(20), -- Fácil, Médio, Difícil
  ativa BOOLEAN DEFAULT TRUE
);
✅ 5. respostas_usuarios
Já está bem estruturada, mas adicione status_resposta.

sql
Copiar
Editar
CREATE TABLE respostas_usuarios (
  id_resposta SERIAL PRIMARY KEY,
  id_usuario INT REFERENCES usuarios(id_usuario),
  id_pergunta INT REFERENCES perguntas(id_pergunta),
  resposta_dada CHAR(1),
  correta BOOLEAN,
  data_hora_resposta TIMESTAMP DEFAULT now(),
  tempo_gasto_segundos INT,
  status_resposta VARCHAR(20) -- 'Acertou', 'Errou', 'Abandonou', 'Tempo Esgotado'
);
✅ 6. sessoes_quiz
Para controle de abandono e início/fim de sessões.

sql
Copiar
Editar
CREATE TABLE sessoes_quiz (
  id_sessao SERIAL PRIMARY KEY,
  id_usuario INT REFERENCES usuarios(id_usuario),
  data_inicio TIMESTAMP DEFAULT now(),
  data_fim TIMESTAMP,
  abandonada BOOLEAN DEFAULT FALSE
);
✅ 7. pontos_usuarios
Tabela para armazenar pontos acumulados.

sql
Copiar
Editar
CREATE TABLE pontos_usuarios (
  id_usuario INT PRIMARY KEY REFERENCES usuarios(id_usuario),
  pontos_totais INT DEFAULT 0,
  pontos_mensais INT DEFAULT 0,
  pontos_anuais INT DEFAULT 0
);
✅ 8. evolucao_usuario
Histórico de desempenho diário/semanal.

sql
Copiar
Editar
CREATE TABLE evolucao_usuario (
  id SERIAL PRIMARY KEY,
  id_usuario INT REFERENCES usuarios(id_usuario),
  data DATE,
  perguntas_respondidas INT,
  acertos INT,
  erros INT,
  abandonos INT,
  tempo_medio_segundos INT
);
✅ 9. log_abandono
Tabela auxiliar para registrar eventos de abandono.

sql
Copiar
Editar
CREATE TABLE log_abandono (
  id SERIAL PRIMARY KEY,
  id_usuario INT REFERENCES usuarios(id_usuario),
  id_pergunta INT REFERENCES perguntas(id_pergunta),
  data_hora TIMESTAMP DEFAULT now()
);
🖥️ Proposta de Telas do Sistema (Front-End / UX)
1. Tela de Login/Cadastro
Cadastro com nome, email, estado, cidade, telefone, sexo.

Login via email + senha.

2. Tela de Seleção de Categoria e Nível
Listar todas as categorias.

Selecionar nível: Fácil / Médio / Difícil.

3. Tela de Quiz
Exibe pergunta + opções.

Temporizador de 20 segundos visível.

Feedback visual de acerto/erro.

Detecção de abandono.

4. Tela de Resultado da Rodada
Número de acertos, erros, abandonos.

Pontos obtidos.

5. Tela de Dashboard Pessoal
Gráficos:

Acertos vs Erros (últimos 7 dias).

Evolução de Pontos.

Tempo médio por resposta.

Indicadores:

Nível atual.

Ranking atual.

Total de perguntas respondidas.

6. Tela de Ranking Geral
Top 10 usuários com mais pontos gerais.

7. Tela de Ranking Mensal
Top 10 do mês atual.

Filtro por categoria opcional.

8. Tela de Histórico
Lista de sessões respondidas.

Datas, categoria, acertos, tempo.

📊 Sistema de Pontuação (Sugerido)
Tipo de Resposta	Pontos Ganhos
Acerto	+10
Erro	0
Tempo Esgotado	0
Abandono	-5 (ou 0)

Cada 100 pontos → Novo nível desbloqueado (configurado na tabela niveis).

Ranking é por soma de pontos corretos.

📈 Relatórios e Consultas-Chave
Contagem de Acertos/Erros/Abandonos
sql
Copiar
Editar
SELECT
  COUNT(*) FILTER (WHERE correta = TRUE) AS acertos,
  COUNT(*) FILTER (WHERE correta = FALSE AND status_resposta IN ('Errou', 'Tempo Esgotado')) AS erros,
  COUNT(*) FILTER (WHERE status_resposta = 'Abandonou') AS abandonos
FROM respostas_usuarios
WHERE id_usuario = :id_usuario;
Ranking Geral
sql
Copiar
Editar
SELECT u.nome, SUM(CASE WHEN r.correta THEN 10 ELSE 0 END) AS pontos
FROM usuarios u
JOIN respostas_usuarios r ON r.id_usuario = u.id_usuario
GROUP BY u.id_usuario
ORDER BY pontos DESC
LIMIT 10;
Ranking Mensal por Categoria
sql
Copiar
Editar
SELECT u.nome, COUNT(*) AS acertos
FROM usuarios u
JOIN respostas_usuarios r ON r.id_usuario = u.id_usuario
JOIN perguntas p ON p.id_pergunta = r.id_pergunta
JOIN categorias c ON c.id_categoria = p.id_categoria
WHERE r.correta = TRUE
  AND c.nome_categoria = 'Matemática'
  AND DATE_TRUNC('month', r.data_hora_resposta) = DATE_TRUNC('month', now())
GROUP BY u.nome
ORDER BY acertos DESC
LIMIT 10;
📋 Resumo Final
Tabelas Propostas (Total: 9)
usuarios

categorias

niveis

perguntas

respostas_usuarios

sessoes_quiz

pontos_usuarios

evolucao_usuario

log_abandono

Telas Propostas (Total: 8)
Login/Cadastro

Seleção de Categoria e Nível

Tela do Quiz

Resultado da Rodada

Dashboard Pessoal

Ranking Geral

Ranking Mensal por Categoria

Histórico do Usuário
