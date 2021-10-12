<h1 align="center">  🔪 Jason CTF - Tryhackme 🗼 

  
![Jason](https://darkside.blog.br/wp-content/uploads/2019/02/03_-1200-%C3%97-674-jason-600x337.png)</h1>


# 📙 Começando a enumeração

Bom, começando a fazer a enumeração, passamos um nmap na maquina. 

![nmap](https://miro.medium.com/max/1800/1*bDQSWkpZ6chLwb3txxJNtQ.png)

Aqui podemos ver, porta 22 SSH e porta 80 HTTP abertas!

Na descrição da sala temos a seguinte descrição: 
Somos a Horror LLC, nos especializamos em terror, mas um dos aspectos mais assustadores de nossa empresa é nosso servidor web front-end. Não podemos lançar nosso site em seu estado atual e nosso nível de preocupação com nossa segurança cibernética está crescendo exponencialmente. Pedimos que você execute um teste de penetração completo e tente comprometer a conta root. Não existem regras para este compromisso. Boa sorte!

É melhor ficar de olho no front-end daqui para a frente. 

Olhando a aplicação web, nos deparamos com a seguinte index:

![index](https://miro.medium.com/max/1800/1*AZ3y980R1brtsld-_jnHBw.png)

Bom, aqui temos uma aba de envio de email e agora temos a informação que foi construido em NodeJS, isso vai ser útil.
Então passando um tempo explorando pela aplicação web, não achamos nenhum outro diretorio e muito menos algum subdominio, vamos ver como ela se comporta quando nós enviamos o vosso e-mail, interceptando com o BurpSuite. 

![email](https://miro.medium.com/max/1800/1*n5n22NyMBbwjA57ru7GlPg.png)

OW! Aqui vemos que toda vez que nós mandamos o e-mail para a aplicação web, ela responde com um Set-cookie! 
Mas como a aplicação determina esses cookies?
Vamos dar uma olhada em parte do código-fonte da página!

![script](https://cdn.discordapp.com/attachments/832336647164526643/897597594010804234/unknown.png)


Olhando o código-fonte, vemos que o cookie é setado como: sessão=foobar+expira+path. 
Vamos descriptografar o cookie que recebemos na requisição anterior. 

![cookie](https://miro.medium.com/max/1800/1*knAbux7d1JuK9Eit9bF3lA.png)

E aqui vemos que o nosso cookie setado é um JSON com email que mandamos!! Aeeeeeeeeeeeee!!!

# 📙 Explorando o NodeJS

Bom, aqui fica claro o que devemos explorar. Podemos fazer um ataque de NodeJS deserialization, você pode aprender mais sobre essa vulnerabilidade aqui: https://opsecx.com/index.php/2017/02/08/exploiting-node-js-deserialization-bug-for-remote-code-execution/

Então, vamos testar!

O payload usado foi esse:
![payload](https://miro.medium.com/max/1800/1*QSNSAfbQL8JwUZ_nByOeUQ.png)
![shell](https://miro.medium.com/max/1800/1*_DPIuyr5sK-Fy8n98Kc3vw.png)

Nos preparamos usando o modulo http.server do python e deixamos o netcat escutando. 

![preparacao](https://miro.medium.com/max/1800/1*z0USzJke2awbInYi9cVp_Q.png)


Depois de tudo preparardo, criptografamos em base64 e enviamos uma requisição para o servidor junto ao cookie com payload criptografado em base64, e pronto!

![revshell](https://miro.medium.com/max/1800/1*xqJowMnVQHaghg9ZkPVRhA.png)

# 📙 O privesc

Agora que temos a reverse shell, e caimos como o usuario dylan, vamos listar algumas de nossas permissões.

![sudolist](https://miro.medium.com/max/1800/1*u-05BtbpNaQcLiNNZlHU_w.png)

Observando o retorno, podemos ver a execução sem senha do npm (módulo instalador de pacotes do NodeJS), então vamos ao gtfobis!

Sudo
If the binary is allowed to run as superuser by sudo, it does not drop the elevated privileges and may be used to access the file system, escalate or maintain privileged access.
Additionally, arbitrary script names can be used in place of preinstall and triggered by name with, e.g., npm -C $TF run preinstall.

TF=$(mktemp -d)

echo '{"scripts": {"preinstall": "/bin/sh"}}' > $TF/package.json

sudo npm -C $TF --unsafe-perm i



Rodando esses comandos conseguimos rootar a maquina e concluir o CTF!

![root](https://miro.medium.com/max/525/1*9qfbxln1Cy0I_sAkNyCSzA.png)

Essa maquina é basicamente isso, tentei passar toda a writeup com a minha visão de como foi jogar essa maquina e meus pensamentos durante o jogo! Boa play, e keep focus, keep hacking! 


(Imagens maramente ilustrativas, pegas do site https://musyokaian.medium.com/jason-tryhackme-walkthrough-e37371eae6e0 pois não printei quando joguei)
