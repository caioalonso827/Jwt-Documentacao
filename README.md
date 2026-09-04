Entendo a frustração! Vamos resolver isso sem estresse. Se os títulos não estão aparecendo bonitinhos ou visualmente formatados no VS Code, é porque você precisa abrir a Visualização (Preview) do Markdown ou salvar o arquivo com a extensão correta.

Como ver os títulos e a formatação no VS Code:

1. Nomeie o arquivo corretamente: Salve o arquivo com o nome README.md ou Guia.md (a extensão .md é obrigatória para o VS Code entender o Markdown).

2. Abra a visualização formatada: No VS Code, com o arquivo aberto, aperte as teclas Ctrl + Shift + V (no Windows) ou Cmd + Shift + V (no Mac).

3. Ou use o ícone do VS Code: No canto superior direito da tela do editor, clique no ícone que parece um livro/página com uma lupa.

Se mesmo assim você quiser o documento completo puro para copiar de novo sem nenhuma interferência de chat, aqui está ele pronto:

Documentação Técnica: Implementação de Autenticação JWT no ASP.NET Core (.NET 8)
Esta documentação apresenta a estrutura de arquivos necessária para implementar autenticação via JSON Web Token (JWT) em uma Web API .NET 8.

1. Instalação do Pacote Nuget
Execute o comando no terminal do projeto para instalar a biblioteca de autenticação Bearer:

Bash
dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer
2. Configuração da Aplicação (Program.cs)
O arquivo Program.cs é responsável por registrar o serviço de autenticação, validar os parâmetros do token e incluir os middlewares de segurança no pipeline.

C#
using System.Text;
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.IdentityModel.Tokens;

var builder = WebApplication.CreateBuilder(args);

// 1. Chave secreta de autenticação (deve ter no mínimo 32 caracteres)
var secretKey = Encoding.ASCII.GetBytes("SUA_CHAVE_SUPER_SECRETA_E_LONGA_COM_32_CARACTERES!");

// 2. Configura a Autenticação por JWT
builder.Services.AddAuthentication(options =>
{
    options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
})
.AddJwtBearer(options =>
{
    options.RequireHttpsMetadata = false;
    options.SaveToken = true;
    options.TokenValidationParameters = new TokenValidationParameters
    {
        ValidateIssuerSigningKey = true,
        IssuerSigningKey = new SymmetricSecurityKey(secretKey),
        ValidateIssuer = false,
        ValidateAudience = false
    };
});

builder.Services.AddControllers();

var app = builder.Build();

// 3. Ativa os Middlewares de Segurança (Respeite esta ordem exata)
app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();

app.Run();
3. Modelo de Transferência de Dados (DTOs/LoginDto.cs)
Classe responsável por capturar o payload da requisição de autenticação na API.

C#
namespace SuaApi.DTOs;

public record LoginDto(string Email, string Senha);
4. Controller de Autenticação (Controllers/AuthController.cs)
Endpoint público responsável por validar as credenciais do usuário e assinar o token JWT.

C#
using System.IdentityModel.Tokens.Jwt;
using System.Security.Claims;
using System.Text;
using Microsoft.AspNetCore.Mvc;
using Microsoft.IdentityModel.Tokens;
using SuaApi.DTOs;

namespace SuaApi.Controllers;

[ApiController]
[Route("api/[controller]")]
public class AuthController : ControllerBase
{
    [HttpPost("login")]
    public IActionResult Login([FromBody] LoginDto dto)
    {
        // Validação de credenciais (Substituir pela busca no banco de dados)
        if (dto.Email != "admin@email.com" || dto.Senha != "123456")
        {
            return Unauthorized(new { mensagem = "E-mail ou senha inválidos." });
        }

        // Construção e assinatura do token JWT
        var tokenHandler = new JwtSecurityTokenHandler();
        var key = Encoding.ASCII.GetBytes("SUA_CHAVE_SUPER_SECRETA_E_LONGA_COM_32_CARACTERES!");

        var tokenDescriptor = new SecurityTokenDescriptor
        {
            Subject = new ClaimsIdentity(new[]
            {
                new Claim(ClaimTypes.Name, dto.Email)
            }),
            Expires = DateTime.UtcNow.AddHours(2),
            SigningCredentials = new SigningCredentials(
                new SymmetricSecurityKey(key),
                SecurityAlgorithms.HmacSha256Signature)
        };

        var token = tokenHandler.CreateToken(tokenDescriptor);

        return Ok(new { token = tokenHandler.WriteToken(token) });
    }
}
5. Controller Protegida (Controllers/TarefasController.cs)
Exemplo de recurso privado que exige o envio prévio do token JWT no cabeçalho Authorization: Bearer <TOKEN>.

C#
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;

namespace SuaApi.Controllers;

[ApiController]
[Route("api/[controller]")]
[Authorize] // Restringe o acesso a requisições autenticadas
public class TarefasController : ControllerBase
{
    [HttpGet]
    public IActionResult Listar()
    {
        return Ok(new[]
        {
            new { id = 1, titulo = "Estudar para o SAEP" },
            new { id = 2, titulo = "Testar rotas no Thunder Client" }
        });
    }
}
6. Como Testar no Thunder Client / Postman
Gere o Token:

Envie uma requisição POST para http://localhost:5000/api/auth/login

Selecione a opção Body -> JSON e envie:

JSON
{
  "email": "admin@email.com",
  "senha": "123456"
}
Copie o valor da propriedade token retornada na resposta.

Acesse o Endpoint Protegida:

Crie uma nova requisição GET para http://localhost:5000/api/tarefas

Vá até a aba Auth (ou Authorization), escolha Bearer Token e cole o token obtido.

Clique em Send. A resposta deve ser o código 200 OK com os dados protegidos.
