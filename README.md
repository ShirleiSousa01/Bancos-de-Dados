# Bancos-de-Dados
Inicio de Bancos de Dados
CREATE TABLE empresas (
id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
razao_social VARCHAR(180) NOT NULL,
nome_fantasia VARCHAR(180),
cnpj VARCHAR(14) UNIQUE,
email VARCHAR(180),
telefone VARCHAR(30),
ativa BOOLEAN NOT NULL DEFAULT TRUE,
criado_em TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE usuários (
id BIGINT GERADO SEMPRE COMO IDENTITY PRIMARY KEY,
empresa_id BIGINT NOT NULL REFERENCES empresas(id),
nome VARCHAR(150) NOT NULL,
email VARCHAR(180) NOT NULL,
senha_hash TEXT NOT NULL,
perfil VARCHAR(30) NOT NULL DEFAULT 'funcionario'
CHECK (perfil IN (
'administrador', 'financeiro', 'vendedor',
'estoquista', 'fiscal', 'funcionario'
)),
ativo BOOLEAN NOT NULL DEFAULT TRUE,
criado_em TIMESTAMPTZ NOT NULL DEFAULT NOW(),
UNIQUE (empresa_id, email),
UNIQUE (empresa_id, id)
);

CREATE TABLE clientes (
id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
empresa_id BIGINT NOT NULL REFERENCES empresas(id),
nome VARCHAR(180) NOT NULL,
documento VARCHAR(14),
email VARCHAR(180),
telefone VARCHAR(30),
endereco TEXT,
criado_em TIMESTAMPTZ NOT NULL DEFAULT NOW(),
UNIQUE (empresa_id, id)
);

CREATE TABLE produtos (
id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
empresa_id BIGINT NOT NULL REFERENCES empresas(id),
codigo VARCHAR(50) NOT NULL,
nome VARCHAR(180) NOT NULL,
descrição TEXT,
preco_venda NUMERIC(14,2) NOT NULL DEFAULT 0
CHECK (preco_venda >= 0),
custo NUMERIC(14,2) NOT NULL DEFAULT 0
CHECK (custo >= 0),
estoque_minimo NUMERIC(14,3) NOT NULL DEFAULT 0
CHECK (estoque_minimo >= 0),
ativo BOOLEAN NOT NULL DEFAULT TRUE,
UNIQUE (empresa_id, codigo),
UNIQUE (empresa_id, id)
);

CRIAR TABELA vendas (
id BIGINT GERADO SEMPRE COMO IDENTIDADE CHAVE PRIMÁRIA,
empresa_id BIGINT NOT NULL REFERENCES empresas(id),
cliente_id BIGINT,
usuario_id BIGINT NOT NULL,
data_venda TIMESTAMPTZ NOT NULL DEFAULT NOW(),
status VARCHAR(20) NOT NULL DEFAULT 'rascunho'
CHECK (status IN (
'rascunho', 'confirmada',
'cancelada', 'faturada'
)),
valor_total NUMERIC(14,2) NOT NULL DEFAULT 0
CHECK (valor_total >= 0),
FOREIGN KEY (empresa_id, cliente_id)
REFERENCES clientes(empresa_id, id),
FOREIGN KEY (empresa_id, usuario_id)
REFERÊNCIAS usuários(empresa_id, id),
UNIQUE (empresa_id, id)
);

CRIAR TABELA itens_venda (
id BIGINT GERADO SEMPRE COMO CHAVE PRIMÁRIA DE IDENTIDADE,
empresa_id BIGINT NOT NULL,
venda_id BIGINT NOT NULL,
produto_id BIGINT NOT NULL,
quantidade NUMERIC(14,3) NOT NULL
CHECK (quantidade > 0),
preco_unitario NUMERIC(14,2) NOT NULL
CHECK (preco_unitario >= 0),
desconto NUMERIC(14,2) NOT NULL DEFAULT 0
CHECK (desconto >= 0),
FOREIGN KEY (empresa_id, venda_id)
REFERENCES vendas(empresa_id, id),
FOREIGN KEY (empresa_id, produto_id)
REFERENCES produtos(empresa_id, id),
CHECK (desconto <= quantidade * preco_unitario)
);

CREATE TABLE movimentoscoes_estoque (
id BIGINT GERADO SEMPRE COMO IDENTITY PRIMARY KEY,
empresa_id BIGINT NOT NULL,
produto_id BIGINT NOT NULL,
usuario_id BIGINT NOT NULL,
tipo VARCHAR(10) NOT NULL
CHECK (tipo IN ('entrada', 'saida', 'ajuste')),
quantidade NUMERIC(14,3) NOT NULL
CHECK (quantidade > 0),
motivo TEXT NOT NULL,
referência VARCHAR(100),
criado_em TIMESTAMPTZ NOT NULL DEFAULT NOW(),
FOREIGN KEY (empresa_id, produto_id)
REFERENCES produtos(empresa_id, id),
FOREIGN KEY (empresa_id, usuario_id)
REFERENCES usuarios(empresa_id, id)
);

CREATE TABLE pedidos_servico (
id BIGINT GERADO SEMPRE COMO IDENTITY PRIMARY KEY,
empresa_id BIGINT NOT NULL REFERENCES empresas(id),
cliente_id BIGINT NOT NULL,
responsavel_id BIGINT NOT NULL,
descricao TEXT NOT NULL,
status VARCHAR(20) NOT NULL DEFAULT 'aberta'
CHECK (status IN (
'aberta', 'em_andamento',
'concluida', 'cancelada'
)),
valor NUMERIC(14,2) NOT NULL DEFAULT 0
CHECK (valor >= 0),
data_abertura TIMESTAMPTZ NOT NULL DEFAULT NOW(),
data_conclusao TIMESTAMPTZ,
FOREIGN KEY (empresa_id, cliente_id)
REFERÊNCIAS clientes(empresa_id, id),
CHAVE ESTRANGEIRA (empresa_id, responsavel_id)
REFERÊNCIAS usuários(empresa_id, id)
);

CREATE TABLE lancamentos_financeiros (
id BIGINT GERADO SEMPRE COMO IDENTITY PRIMARY KEY,
empresa_id BIGINT NOT NULL REFERENCES empresas(id),
cliente_id BIGINT,
venda_id BIGINT,
tipo VARCHAR(10) NOT NULL
CHECK (tipo IN ('receber', 'pagar')),
descrição TEXT NOT NULL,
valor NUMERIC(14,2) NOT NULL CHECK (valor > 0),
vencimento DATE NOT NULL,
pagamento DATE,
status VARCHAR(15) NOT NULL DEFAULT 'aberto'
CHECK (status IN (
'aberto', 'pago', 'parcial', 'cancelado'
)),
FOREIGN KEY (empresa_id, cliente_id)
REFERENCES clientes(empresa_id, id),
FOREIGN KEY (empresa_id, venda_id)
REFERÊNCIAS vendas(empresa_id, id)
);

CRIAR TABELA documentos_fiscais (
id BIGINT GERADO SEMPRE COMO CHAVE PRIMÁRIA DE IDENTIDADE,
empresa_id BIGINT NOT NULL REFERENCES empresas(id),
venda_id BIGINT,
tipo VARCHAR(30) NOT NULL,
numero VARCHAR(50),
chave_acesso VARCHAR(60),
status VARCHAR(20) NOT NULL DEFAULT 'pendente'
CHECK (status IN (
'pendente', 'autorizado',
'rejeitado', 'cancelado'
)),
criado_em TIMESTAMPTZ NOT NULL DEFAULT NOW(),
FOREIGN KEY (empresa_id, venda_id)
REFERÊNCIAS vendas(empresa_id, id)
);

CREATE TABLE auditorias (
id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
empresa_id BIGINT NOT NULL REFERENCES empresas(id),
usuario_id BIGINT,
acao VARCHAR(100) NOT NULL,
tabela_afetada VARCHAR(100) NOT NULL,
registro_id BIGINT,
detalhes JSONB,
criado_em TIMESTAMPTZ NOT NULL DEFAULT AGORA(),
CHAVE ESTRANGEIRA (empresa_id, usuario_id)
REFERÊNCIAS usuarios(empresa_id, id)
);

-- Índices para consultas frequentes
CRIAR ÍNDICE idx_vendas_data
ON vendas (empresa_id, data_venda);

CRIAR ÍNDICE idx_estoque_produto
ON movimentacoes_estoque (empresa_id, produto_id, criado_em);

CRIAR ÍNDICE idx_financeiro_vencimento
ON lancamentos_financeiros (empresa_id, status, vencimento);

CRIAR ÍNDICE idx_servicos_status
ON ordens_servico (empresa_id, status);
