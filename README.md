# LithNetPasswordProtection
Repositório de implementação do Lithnet em domínio windows para validar senhas fracas em tempo real ou encontradas no diretório do HIBP (Have i been pwned) haveibeenpwned.com

 Pré-requisitos
- Windows Server 2019  
- Active Directory Domain Services  
- Lithnet Password Protection  
- API Key válida do HaveIBeenPwned  
- PowerShell 5.1+

## Iniciaremos realizando uma instalação limpa do active directory, para laboratório irei utilizar uma imagem do Windows server 2019, porém foi validado que a ferramenta funciona a partir da versão 2012 R2 até a 2025 e o software VirtualBox, com as configurações básicas descritas nas imagens abaixo.

<img width="790" height="350" alt="image" src="https://github.com/user-attachments/assets/57c46809-24b4-480c-9314-5ea814a0d20a" />
<img width="770" height="172" alt="image" src="https://github.com/user-attachments/assets/3e093d50-5e6a-4677-96d6-c22109e324ee" />
<img width="789" height="138" alt="image" src="https://github.com/user-attachments/assets/3de95f1c-1f3f-40c5-b9ce-002cdc072c73" />
<img width="782" height="259" alt="image" src="https://github.com/user-attachments/assets/93d4fd2d-32d1-47e4-8a30-a7f79cc346bc" />

### Utilizaremos a versão Datacenter (Desktop Experience) para facilitar a utilização dos comandos e instalação da ferramenta.
 #### Realizar uma instalação padrão e depois renomear a máquina para o nome que ela deverá ficar, neste caso optei pelo nome ServerTestes19
 #### <img width="1024" height="857" alt="image" src="https://github.com/user-attachments/assets/9d75c685-6a24-4796-be9e-b3bcb5cb9f1d" />
 #### Agora devemos instalar a role Domain Services
 <img width="784" height="563" alt="image" src="https://github.com/user-attachments/assets/2c20dc34-6d89-4996-bcf6-957449566293" />
 #### Não é necessário adicionar nenhuma feature adicional para esta instalação.
 #### Depois disso iremos tornar nosso server em um Controlador de Dominio promovendo ele ao nosso domínio testes.local
 #### Para isso devemos lembrar da senha cadastrada no momento da criação da máquina.
 <img width="1023" height="733" alt="image" src="https://github.com/user-attachments/assets/2e7494f5-cfae-466e-8184-b0dc469d0392" />
 #### Adicionaremos uma nova árvore e colocaremos nosso domínio (caso tenha algum domínio válido ou for colocar em produção usar o domínio correto)
 <img width="1025" height="731" alt="image" src="https://github.com/user-attachments/assets/73ca8940-6bca-493b-a873-2b63fb36b43d" />
 #### Nesse passo devemos selecionar o nível operacional do nosso domínio, deixarei como Server 2016 para demonstração, porém podemos reduzir caso necessário
 <img width="765" height="561" alt="image" src="https://github.com/user-attachments/assets/f0b78e4f-45d8-44fd-bdcd-184774b12fa9" />
 #### Após reiniciada a máquina devemos instalar o Lithnet com uma instalação padrã NNF(Next,Next,Finish)
 #### Realizamos o download através do link (https://packages.lithnet.io/win/password-protection/v1.1/x64/stable)
 #### Lembrando que devemos também instalar o VCRedist juntamente
 #### Após isso caso não sejam encontrados os templates do windows presentes na pasta (C:\Windows\PolicyDefinitions) devemos realizar o download dos mesmos através do link (https://learn.microsoft.com/en-au/troubleshoot/windows-client/group-policy/create-and-manage-central-store) e copiá-los para dentro da pasta (C:\Windows\SYSVOL\domain\Policies\PolicyDefinitions) e (C:\Windows\SYSVOL\domain\Policies\PolicyDefinitions\en-US) conforme o print
 <img width="589" height="115" alt="image" src="https://github.com/user-attachments/assets/e481eabd-f34a-4ca5-ba43-4befa34e6677" />
 <img width="654" height="106" alt="image" src="https://github.com/user-attachments/assets/97c9f1aa-0824-489e-a4f5-720230e2d3a5" />

 #### Feito isso iremos configurar nossa GPO, para isso vamos abrir o gerenciador com o comando Windows+R e digitar gpmc.msc
 #### Podemos desabilitar as políticas de senhas padrão ou deixar ativado, depende da utilização.
 #### Caso desejar deixar como está devemos criar uma nova.
 #### Clicar com o botão direito no campo Group Policy Objects -> New
 <img width="753" height="532" alt="image" src="https://github.com/user-attachments/assets/0cd2616c-d446-439b-b77d-c6748f8e136d" />

 #### Devemos vinculá-la a UO Domain Controllers para surtir efeito no domínio.
 
 #### Inserir o nome para a nova política;
 <img width="387" height="183" alt="image" src="https://github.com/user-attachments/assets/5514a06c-c067-4958-a2eb-011ed071bacf" />

 #### Clicar com o botão direito e editar;
 <img width="249" height="50" alt="image" src="https://github.com/user-attachments/assets/dd941128-091b-4afe-a72b-f721db75da6a" />

 #### Expandir a política até chegar na parte Computer Configuration -> Policies -> Administrative Templates -> Lithnet -> Password Protection for Active Directory -> Default Policy
 ##### Configurar as opções desejadas;
 <img width="794" height="583" alt="image" src="https://github.com/user-attachments/assets/32cbc744-d4ce-4962-9f53-b8f5963997cf" />

 #### Para o laboratório foram configuradas as seguintes políticas;
 #### Minimum password lenght (Minimo de caracteres exigidos);
 #### Reject passwords that contain the user’s account name(enable);
 #### Reject passwords that contain the user’s display name(enable);
 #### Reject normalized passwords found in the compromissed password store(enable);
 #### Reject normalized passwords found in the banned word store(enable);
  
 #### Isto faz com que seja setada uma senha com os seguintes requisitos:
 #### Não utilizar no campo da senha o nome do usuário;
 #### Rejeitar senhas encontradas na loja;
 #### Rejeita palavras encontradas na loja;e
 #### Tamanho mínimo de 15 caracteres.
 #### Feitas as configurações podemos acessar nossa store onde irá ficar armazenado as nossas palavras proibidas e adicioná-las, para isso devemos abrir o powershell como administrador, Windows+R digitar powershell -> segurar CTRL+SHIFT e dar enter;
 #### Importar o módulo do Lithnet (este passo deverá ser feito todas as vezes que necessitarmos adicionar palavras na loja);
     - Import-Module LithnetPasswordProtection
 #### Configurar o caminho da loja;  
     - Set-PasswordFilterConfig -StorePath C:\Program Files\Lithnet\Active Directory Password Protection\Store
 #### Abrir a loja (Este passo também deverá ser realizado sempre que adicionarmos palavras na loja); 
     - Open-Store -Path 'C:\Program Files\Lithnet\Active Directory Password Protection\Store\'
 <img width="1323" height="329" alt="image" src="https://github.com/user-attachments/assets/05b7b7a2-65dd-48b1-85f8-7fab29ab830a" />

 #### Adicionar as palavras que desejamos bloquear; 
     - Add-BannedWord -Value 'teste'  #Isso irá banir a palavra teste e suas variações do uso de senhas, EX: bloqueará senhas tipo teste123, teste@123 t3st3 t3$t3.
 #### Realizados estes passos podemos testar se a senha foi bloqueada conforme o esperado, para isso dentro do powershell ainda usaremos o cmdlet 'Test-IsBannedWord -Value 'teste'
 #### <img width="1304" height="385" alt="image" src="https://github.com/user-attachments/assets/174067ac-9160-4ae1-9821-fb3fd0be6350" />

