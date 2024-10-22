# Manual de Backup e Restauração do PostgreSQL

> **Aviso de Responsabilidade:** Este manual é fornecido como uma orientação técnica. A total responsabilidade pelo procedimento é de quem o está executando. Certifique-se de entender completamente cada etapa e, caso haja dúvidas, consulte a documentação oficial do PostgreSQL para orientações detalhadas e atualizadas.

## 1. Backup de Banco de Dados

### 1.1 Procedimento de Backup com `pg_dump`

Para realizar um backup completo do banco de dados utilizando o formato de diretório, que permite o uso de múltiplas threads, siga os passos abaixo:

1. **Comando para Backup com `pg_dump`**:

   ```bash
   pg_dump -U nome_do_usuario -h localhost -Fd -j 4 -f /caminho/do/backup/nome_do_diretorio nome_da_base_de_dados
   ```

   - `-U`: Nome do usuário do PostgreSQL.
   - `-h`: Host onde o servidor PostgreSQL está rodando (exemplo: localhost).
   - `-Fd`: Define o formato de diretório para o backup.
   - `-j 4`: Número de threads paralelas para o backup (substituir pelo número de CPUs disponíveis).
   - `-f`: Caminho para o diretório onde os arquivos de backup serão salvos.
   - `nome_da_base_de_dados`: Nome do banco de dados a ser salvo.

2. **Verificar o Backup Criado**:

   O comando acima criará múltiplos arquivos dentro do diretório especificado. Certifique-se de que o diretório e os arquivos foram criados corretamente.

### 1.2 Compressão e Validação com MD5 (Opcional)

Caso opte por compactar o diretório de backup para transferência, recomendamos validar a integridade com checksums MD5. Siga as etapas abaixo:

1. **Compactar o Diretório de Backup**:

   ```bash
   tar -czvf backup.tar.gz /caminho/do/backup/nome_do_diretorio
   ```

2. **Gerar Checksum MD5 para o Arquivo Compactado**:

   ```bash
   md5sum backup.tar.gz > backup.md5
   ```

3. **Verificar Checksum MD5 após a Transferência**:

   No servidor de destino, verifique a integridade do arquivo:

   ```bash
   md5sum -c backup.md5
   ```

   Isso garantirá que o arquivo compactado foi transferido corretamente e sem corrupção.

---

## 2. Restauração de Banco de Dados

### 2.1 Procedimento de Restauração com `pg_restore`

1. **Comando para Restauração com `pg_restore`**:

   ```bash
   pg_restore -U nome_do_usuario -h localhost -d nome_da_base_de_dados -j 4 /caminho/do/backup/nome_do_diretorio
   ```

   - `-U`: Nome do usuário do PostgreSQL.
   - `-h`: Host onde o servidor PostgreSQL está rodando.
   - `-d`: Nome da base de dados onde o backup será restaurado.
   - `-j 4`: Número de threads paralelas para a restauração (substituir pelo número de CPUs disponíveis).
   - `/caminho/do/backup/nome_do_diretorio`: Caminho para o diretório de backup.

2. **Verificar a Restauração**:

   Certifique-se de que todos os dados e esquemas foram restaurados corretamente na base de dados de destino.

---

## 3. Procedimento Opcional: Transferência para Outro Servidor

Caso o backup precise ser transferido para outro servidor, sugerimos gerar o backup diretamente no servidor de destino para maior eficiência. No entanto, se a transferência for necessária, siga um dos métodos abaixo:

### 3.1 Transferência do Backup com `scp` (Opcional)

```bash
scp -r /caminho/do/backup/nome_do_diretorio usuario@ip_da_maquina_remota:/caminho/de/destino/
```

- `-r`: Copia recursivamente todos os arquivos do diretório de backup.
- `usuario`: Usuário da máquina de destino.
- `ip_da_maquina_remota`: Endereço IP da máquina de destino.
- `/caminho/de/destino/`: Caminho de destino na máquina remota.

### 3.2 Transferência de Arquivo Compactado (Opcional)

Se optar por compactar o backup:

1. **Compactar o Diretório de Backup**:

   ```bash
   tar -czvf backup.tar.gz /caminho/do/backup/nome_do_diretorio
   ```

2. **Transferir o Arquivo Compactado com `scp`**:

   ```bash
   scp backup.tar.gz usuario@ip_da_maquina_remota:/caminho/de/destino/
   ```

3. **Descompactar na Máquina Remota**:

   ```bash
   tar -xzvf /caminho/de/destino/backup.tar.gz -C /caminho/de/extracao/
   ```

---

## 4. Auxílio em Caso de Erros durante o Restore

### 4.1 Erro de Autenticação: "Password authentication failed for user"

#### 4.1.1 Verificar Configuração do `pg_hba.conf`

1. Abra o arquivo `pg_hba.conf`:

   ```bash
   nano /etc/postgresql/[versao]/main/pg_hba.conf
   ```

2. Verifique se há uma linha semelhante a:

   ```
   local   all   postgres   md5
   ```

   Certifique-se de que o método de autenticação é `md5` para permitir a autenticação por senha.

3. Reinicie o serviço PostgreSQL:

   ```bash
   sudo systemctl restart postgresql
   ```

#### 4.1.2 Redefinir a Senha do Usuário `postgres`

1. Acesse o PostgreSQL como o usuário `postgres`:

   ```bash
   sudo -i -u postgres
   psql
   ```

2. Redefina a senha:

   ```sql
   ALTER USER postgres PASSWORD 'nova_senha';
   ```

3. Saia do `psql`:

   ```sql
   \q
   ```

### 4.2 Verificar a Conexão com `psql`

Certifique-se de que consegue conectar usando o comando:

```bash
psql -U postgres -h localhost -d nome_da_base_de_dados
```

Se conseguir conectar, a autenticação está funcionando corretamente.

---

## 5. Conclusão

Este manual fornece um guia técnico detalhado para realizar backups e restaurações eficientes no PostgreSQL utilizando o `pg_dump` e `pg_restore`. Ele também aborda problemas comuns de autenticação durante a restauração e recomendações para validação de integridade do backup com checksums MD5.
