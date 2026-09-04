# Documentação de Implementação JWT (.NET 8)

## 1. Arquivo: `Program.cs`
Responsável pela inicialização da aplicação, registro dos serviços do JWT e ativação dos middlewares de segurança.

```csharp
using System.Text;
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.IdentityModel.Tokens;

var builder = WebApplication.CreateBuilder(args);

// 1. Chave secreta de autenticação (mínimo de 32 caracteres)
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

// 3. Ativa os Middlewares de Segurança (A ordem importa!)
app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();

app.Run();




## 3. Arquivo: Controllers/AuthController.cs
## Controller responsável pela rota pública de login e pela emissão do token JWT.

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
        // Validação de exemplo (Substituir pela busca no banco de dados)
        if (dto.Email != "admin@email.com" || dto.Senha != "123456")
        {
            return Unauthorized(new { mensagem = "E-mail ou senha inválidos." });
        }

        // Configuração e geração do Token JWT
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
