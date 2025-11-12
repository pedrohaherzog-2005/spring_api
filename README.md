# Pesquisa sobre autenticação em Api's

## 1. O que é autenticação HTTP Basic?  
A autenticação HTTP Basic (ou “Basic Auth”) é um esquema de autenticação para HTTP em que o cliente envia no cabeçalho `Authorization` as credenciais no formato `username:password`, codificadas em Base64.  
Embora simples e suportado por muitos servidores/clients, apresenta limitações importantes: a codificação Base64 **não** oferece criptografia, e portanto sem HTTPS as credenciais podem ser interceptadas. Além disso, exige que as credenciais sejam enviadas em cada requisição ou que o cliente as “cacheie”.  
Uso típico: aplicações simples, ferramentas internas ou APIs muito básicas.

## 2. Qual a diferença entre autenticação e autorização?  
- **Autenticação:** Verificação da identidade de um usuário ou sistema. Em outras palavras: “quem é você?”.  
- **Autorização:** Após a autenticação, determinação do que esse usuário autenticado pode fazer ou a quais recursos tem acesso: “o que você pode fazer?”.  
A autenticação normalmente precede a autorização. Uma vez identificada a identidade, o sistema pode aplicar regras de acesso/permissão.

## 3. Como armazenar usuários no banco de dados em vez de memória?  
Para persistência de usuários (em vez de armazenar em memória durante execução):  
- Crie uma tabela no banco (ex: `users`) com colunas como `id`, `username`, `password_hash`, `email`, `roles`, `is_active`, timestamps etc.  
- Armazene senhas **criptografadas** (usar bcrypt, Argon2, etc) e nunca em texto claro.  
- Use ORM ou JDBC para persistir e recuperar usuários.  
- Para autorização, mantenha tabelas de roles/permissions (ex: `roles`, `user_roles`) em relação many-to-many.  
- No login: busque o usuário por `username`, compare a senha inserida (após hash) com a senha armazenada, verifique se ativo etc, e então crie sessão ou token conforme arquitetura.

## 4. Como implementar autenticação com JWT?  
Implementar autenticação com JWT envolve:  
1. Criar endpoint de login que recebe credenciais (usuário+senha). 
2. Validar credenciais; se válidas, gerar um JWT contendo claims (como `sub = userId`, `roles`, `exp = prazo de expiração`, etc).  
3. Retornar o JWT ao cliente.  
4. Cliente envia, nas requisições subsequentes, o token no cabeçalho, geralmente `Authorization: Bearer <token>`.  
5. No servidor, antes de servir o recurso protegido, validar o token: ver assinatura, expiração, extrair claims, montar contexto de segurança (usuário + roles).  
6. Opcionalmente, implementar refresh token para tokens mais longos, blacklist para revogação, ou políticas de expiração curta + renovação para controlar segurança.  
7. Tomar cuidado com onde armazenar o token (localStorage, cookie HttpOnly, etc) e com vetores de ataque como XSS ou CSRF dependendo do caso.

## 5. O que é CSRF e por que ele foi desabilitado aqui?  
O **CSRF** é um tipo de ataque onde um navegador autenticado num site é enganado a enviar requisições não intencionadas para esse site (ou outro) em que o usuário já está autenticado (normalmente via cookie de sessão enviado automaticamente pelo navegador). O site malicioso explora essa confiança para executar ações em nome do usuário.  
A proteção CSRF normalmente é necessária quando a autenticação depende de **cookies de sessão** que são enviados automaticamente pelo navegador.  
Se a aplicação estiver funcionando com uma API stateless que utiliza tokens JWT enviados explicitamente no cabeçalho `Authorization` (e não cookies de sessão automáticos), então o risco clássico de CSRF é muito reduzido, e muitos frameworks optam por **desabilitar** verificação CSRF nesse contexto. Ou seja, foi desabilitado porque o mecanismo de autenticação/token usado não era vulnerável da forma tradicional de CSRF.