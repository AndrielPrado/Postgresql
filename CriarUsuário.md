### 1. **Acessar o PostgreSQL como um usuário com permissões de superusuário:**
   ```bash
   sudo -u postgres psql
   ```
   Ou, se você já estiver autenticado como usuário `postgres`, basta rodar:
   ```bash
   psql
   ```

### 2. **Criar o novo usuário (ou role):**
   Por convenção, recomenda-se o uso de `roles` em PostgreSQL.
   ```sql
   CREATE ROLE nome_do_usuario WITH LOGIN PASSWORD 'sua_senha';
   ```

### 3. **Revogar quaisquer permissões padrão para segurança:**
   ```sql
   REVOKE ALL ON DATABASE nome_do_banco FROM nome_do_usuario;
   ```

### 4. **Conceder permissões de conexão no banco de dados:**
   ```sql
   GRANT CONNECT ON DATABASE nome_do_banco TO nome_do_usuario;
   ```

### 5. **Configurar o esquema padrão para o usuário (opcional, mas recomendável):**
   ```sql
   ALTER USER nome_do_usuario SET search_path TO public;
   ```

### 6. **Conceder permissões de somente leitura para todas as tabelas do banco de dados:**
   Conceda o acesso somente de leitura às tabelas de um esquema específico (como o esquema padrão `public`):
   ```sql
   GRANT USAGE ON SCHEMA public TO nome_do_usuario;
   GRANT SELECT ON ALL TABLES IN SCHEMA public TO nome_do_usuario;
   ```

### 7. **Ajustar permissões para tabelas futuras (opcional, mas recomendável):**
   Para garantir que tabelas futuras também tenham permissões de leitura automaticamente:
   ```sql
   ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO nome_do_usuario;
   ```

### 8. **Sair do PostgreSQL:**
   ```sql
   \q
   ```

### Resumo dos comandos:

```sql
CREATE ROLE nome_do_usuario WITH LOGIN PASSWORD 'sua_senha';
REVOKE ALL ON DATABASE nome_do_banco FROM nome_do_usuario;
GRANT CONNECT ON DATABASE nome_do_banco TO nome_do_usuario;
ALTER USER nome_do_usuario SET search_path TO public;
GRANT USAGE ON SCHEMA public TO nome_do_usuario;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO nome_do_usuario;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO nome_do_usuario;
```

Este procedimento cria um usuário com permissões limitadas de somente leitura no banco de dados e garante que novas tabelas também herdarão essa permissão.